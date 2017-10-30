---
layout: post
title:  "Unattended authentication against the Microsoft Graph API from PowerShell"
date:   2017-10-30 05:00:00 +0200
comments: true
categories: Azure,Graph,API
---

## Background - Microsoft Graph

*You can use the Microsoft Graph API to interact with the data of millions of users in the Microsoft cloud. Use Microsoft Graph to build apps for organizations and consumers that connect to a wealth of resources, relationships, and intelligence, all through a single endpoint: https://graph.microsoft.com.*

![alt](/images/2017-10-28_microsoft_graph_01.png){:height="293px" width="600px"}  
*Image credit: Microsoft*

You can read more about Microsoft Graph on developer.microsoft.com:  
- [Microsoft Graph](https://developer.microsoft.com/en-us/graph/)  
- [Overview of Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/concepts/overview)

In this article, we are going to work with Intune in Microsoft Graph - although the authentication concept is the same for Microsoft Graph in general.

*The Microsoft Graph API for Intune enables programmatic access to Intune information for your tenant; the API performs the same Intune operations as those available through the Azure Portal.*

*Intune provides data into the Microsoft Graph in the same way as other cloud services do, with rich entity information and relationship navigation.  Use Microsoft Graph to combine information from other services and Intune to build rich cross-service applications for IT professionals or end users.     
Here is an example of how you can determine whether an application is installed on a user's device:*

-  *Get from Azure Active Directory a list of devices registered to a user:*  
https://graph.microsoft.com/beta/users/{user}/ownedDevices 
- *Then view the list of applications for your tenant:*  
https://graph.microsoft.com/beta/deviceAppManagement/mobileApps
- *Take the ID from the application and determine the installation state for the application (and therefore user):*
https://graph.microsoft.com/beta/deviceAppManagement/mobileApps/{id}/deviceStatuses/

You can read more in the [Intune documentation for Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/api-reference/beta/resources/intune_graph_overview).

## Challenge

When working with automation, there is often a need to perform unattended authentication. This can be from a scheduled task on a server or in an automation service such as Azure Automation.

In our example scenario, we want to do some automation against Intune. Optimally, we would use an official PowerShell module from Microsoft for this task. However, such module does not exist as of October 2017. There is a UserVoice suggestion called *[Add PowerShell support to manage the service](https://microsoftintune.uservoice.com/forums/291681-ideas/suggestions/8363319-add-powershell-support-to-manage-the-service)* which has 773 votes as of this writing, and a lot of comments from people wanting this.

Since Intune now has migrated to Azure and is using the Microsoft Graph API, we can leverage that to perform our automation tasks.

Providing a username and password in the form of a PowerShell credential object is not sufficient in this scenario, as the API is based on OAuth 2.0.

There are a lot of examples online on how to get an access token for authenticating to the API, but most of these will leverage Azure Active Directory Authentication Library (ADAL) forms based login and thus work only for interactive sessions:

![alt](/images/2017-10-28_microsoft_graph_09.png)

A different approach would be to create a web application registration in the Microsoft App Registration Portal (instructions: [Add an application to Azure Active Directory](https://docs.microsoft.com/nb-no/azure/active-directory/develop/active-directory-integrating-applications#adding-an-application)), which would work fine for authenticating against the Microsoft Graph API. We would also be able to perform tasks such as retrieve users and similar tasks against Azure Active Directory. However, if we look at the Intune specific APIs ([example](https://developer.microsoft.com/en-us/graph/docs/api-reference/beta/api/intune_devicefe_manageddevice_get)), they do not support applications (per October 2017):
![alt](/images/2017-10-28_microsoft_graph_10.png){:height="166px" width="600px"}

Hence, you would get a "Bad request" error if trying to use a registered web application against the Intune APIs.

## Solution

The high level steps is documented in the [Get access without a user](https://developer.microsoft.com/en-us/graph/docs/concepts/auth_v2_service) section in the Microsoft Graph documentation:

1. Create a native application registration in Azure Active Directory

2. Configure application permissions for Microsoft Graph

3. Get administrator consent

4. Get an access token

5. Use the access token to call Microsoft Graph

# Walkthrough

1. Add an application to Azure Active Directory  

Detailed instructions is available in the [documentation](https://docs.microsoft.com/nb-no/azure/active-directory/develop/active-directory-integrating-applications#adding-an-application).

![alt](/images/2017-10-28_microsoft_graph_02.png)  

- **Name**: A friendly name - not used from the application/authentication process  
- **Application type:** Native (a client application that are installed locally on a device - in this scenario it is the ADAL client libraries which is embedded in the AzureAD PowerShell module)  
- **Redirect URI:** Must be set to the well-known URI for Azure PowerShell *urn:ietf:wg:oauth:2.0:oob* (the URI is used by Azure AD to return token responses)

![alt](/images/2017-10-28_microsoft_graph_03.png)  

When the application is created, notice the Application ID for the newly created application, this is the *ClientId* we will need to use in step 4.

2. Configure application permissions for Microsoft Graph  

Next, go to *Required permissions* in the application's Settings:

![alt](/images/2017-10-28_microsoft_graph_04.png){:height="224px" width="600px"}  

Click *Add* and select *Microsoft* in the *Select an API* blade

In the *Select permissions*  blade, select the permissions necessary for the task(s) you are going to automate:

![alt](/images/2017-10-28_microsoft_graph_05.png){:height="212px" width="600px"}  



Click *Select* and *Done*

3. Get administrator consent  

 Some permissions always require a tenant administrator's consent. Whether or not a permission requires admin consent is determined by the developer that published the resource, and can be found in the documentation for the resource. In this scenario, administrator consent is required for the permissions we selected in the previous step, hence we need to press the *Grant Permissions* button:
![alt](/images/2017-10-28_microsoft_graph_06.png)

And then confirm by pressing *Yes*:
![alt](/images/2017-10-28_microsoft_graph_07.png)

4. Get an access token  

There is no need to re-invent the wheel, as the Microsoft Graph GitHub account has a repository containing [PowerShell Intune Samples](https://github.com/microsoftgraph/powershell-intune-samples) which we can use as a starting point :  

![alt](/images/2017-10-28_microsoft_graph_08.png){:height="97px" width="600px"}

In the [Authentication](https://github.com/microsoftgraph/powershell-intune-samples/tree/master/Authentication) folder, there is a sample [Auth_From_File.ps1](https://github.com/microsoftgraph/powershell-intune-samples/blob/master/Authentication/Auth_From_File.ps1) script.

Although the script shows how to get an access token using a username and password stored in clear-text in a file, we can modify the function to accept a regular PowerShell credential object instead. The 
[Get-MSGraphAuthenticationToken](
https://github.com/janegilring/PSCommunity/blob/master/Azure/Active%20Directory/Get-MSGraphAuthenticationToken.ps1) function is a modified version with that modification as well as some additional error handling.

Here is an example on how to use this function to generate an access token:
{% gist 23ef96d6cb31e5dbd8454e4724ffc440 %}

Sample output:  
![alt](/images/2017-10-28_microsoft_graph_12.png){:height="201px" width="600px"}

5. Use the access token to call Microsoft Graph  

Now that we have obtained a valid token, we are ready to consume it while performing an action against the Microsoft Graph API.

In PowerShell, we can leverage the [Invoke-RestMethod](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-restmethod) cmdlet to send HTTPS requests to the Microsoft Graph API.

Next, we should look at the documentation to know which API URLs is available. The best place to start for Intune is the [Working with Intune in Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/api-reference/beta/resources/intune_graph_overview) site.

A few examples is shown on the overview page:
![alt](/images/2017-10-28_microsoft_graph_22.png){:height="206px" width="600px"}

On the navigation menu on the left, there is also topics for each aspect of the service such as *Manage apps* and *Device configuration*.

Let us try to managed devices for a specific user:  

![alt](/images/2017-10-28_microsoft_graph_23.png){:height="709px" width="600px"}

Notice that we specified the token we created as the header for the request.

It might be easier to look at examples, so let us go back to the [PowerShell Intune Samples](https://github.com/microsoftgraph/powershell-intune-samples) and find a sample for the task we are going to perform. In this example, we want to list Intune devices for a specific user and perform a remote action against it. Thus, we can look at the samples in the folder [ManagedDevices](https://github.com/microsoftgraph/powershell-intune-samples/tree/master/ManagedDevices).

As we can see in the script samples [GetManagedDevices_Get.ps1](https://github.com/microsoftgraph/powershell-intune-samples/blob/master/ManagedDevices/ManagedDevices_Get.ps1) and [Invoke_DeviceAction_Set.ps1](https://github.com/microsoftgraph/powershell-intune-samples/blob/master/ManagedDevices/Invoke_DeviceAction_Set.ps1), there is already a Get-AuthToken function embedded which leverages interactive logon. Thus, we need to make a customized version of these functions as well:

- [Get-MSGraphAzureADUser](https://github.com/janegilring/PSCommunity/blob/master/Azure/Active%20Directory/Get-MSGraphAzureADUser.ps1)
- [Get-MSGraphIntuneUserDevice](https://github.com/janegilring/PSCommunity/blob/master/Azure/Intune/Get-MSGraphIntuneUserDevice.ps1)
- [Invoke-MSGraphIntuneDeviceAction](https://github.com/janegilring/PSCommunity/blob/master/Azure/Intune/Invoke-MSGraphIntuneDeviceAction.ps1)

Initially, the plan for this article was to provide the Get-MSGraphAuthenticationToken function and a couple of example functions for working with the Intune service via the Microsoft Graph API. However, at this point, I decided to create a PowerShell module and publish it to the [PowerShell Gallery](https://www.powershellgallery.com/packages/MSGraphIntuneManagement) to make it easy to consume. The module is also meant to bridge the gap and be a starting point for those who want to use PowerShell to administer the Intune service until an official Intune PowerShell module is provided by Microsoft. The module is called MSGraphIntuneManagement and can be installed using the following command from the PowerShellGet module:

*Install-Module -Name MSGraphIntuneManagement*

If you do not have the option to leverage the PowerShellGet module, manual installation instructions as well as information about all requirements is available in the [GitHub repository](https://github.com/janegilring/MSGraphIntuneManagement) for the module.

The initial version of the module contains the beforementioned functions, but it should be easy to extend it further based on the [PowerShell Intune Samples](https://github.com/microsoftgraph/powershell-intune-samples) from Microsoft.

Installation and sample usage of the MSGraphIntuneManagement PowerShell module:

{% gist 6e437a56b5ae9fc42a37e422b666dcb3 %}

Shortly after invoking the device action, the following could be seen on the device:

![alt](/images/2017-10-28_microsoft_graph_11.png){:height="511px" width="600px"}

**Azure Automation example**

Let us close up by looking at an unattended scenario by setting up a runbook in Azure Automation with the necessary prerequisites.

First of all, you will need an [Azure subscription](https://azure.microsoft.com/en-us/free/) and an [Automation account](https://docs.microsoft.com/en-us/azure/automation/automation-create-standalone-account).

For this demo, I have created a new Azure Automation account called *Intune-Management*.  
The first step is to import the necessary modules, which for this demo is the AzureAD and the MSGraphIntuneManagement modules:
![alt](/images/2017-10-28_microsoft_graph_14.png){:height="681px" width="600px"}

You can compress a local module to a zip-file and upload it by using the *Add module* button. However, when the modules we need is available on the PowerShell Gallery, it is more convenient to use the *Browse gallery* button and search for them there. When found we can simply click the *Import* button:
![alt](/images/2017-10-28_microsoft_graph_13.png){:height="460px" width="600px"}

The next we need is a credential asset for authenticating to the Microsoft Graph API from runbooks. Go to *Credentials*, click *Add a credential* and specify a service account with permissions to the target Intune tenant:
![alt](/images/2017-10-28_microsoft_graph_15.png){:height="623px" width="600px"}

The steps is similar for adding variables for use within runbooks. Here we add the client id for the Azure AD application we created beforehand which has the necessary permissions such as Intune Device Management:  
![alt](/images/2017-10-28_microsoft_graph_16.png){:height="605px" width="600px"}

The last step is to add a new runbook to perform the task we want to accomplish. In this demo we are creating a runbook for offboarding users from our organization, which will perform a *RemoveCompanyData* on mobile devices and a *FactoryReset* on other device types. Go ahead and create a runbook of the type *PowerShell*:
![alt](/images/2017-10-28_microsoft_graph_21.png)

The contents of the runbook is the following:  

{% gist c243f0085f2f4bb1212be2da296e48c9 %}

When the runbook is created, we can start it and supply a value for the UserPrincipalName for a demo user which has a Windows 10 device registered in Intune:

![alt](/images/2017-10-28_microsoft_graph_17.png)

If all goes well, we should see no errors logged and an Output similar to this:  

![alt](/images/2017-10-28_microsoft_graph_18.png)

A few seconds after the runbook finished, the demo user`s device was restarting:  
![alt](/images/2017-10-28_microsoft_graph_19.png){:height="511px" width="600px"}

Then the reset process was started:  
![alt](/images/2017-10-28_microsoft_graph_20.png){:height="508px" width="600px"}

## Summary

In this article we have looked at what capabilites the Microsoft Graph API can provide. We have also looked at various options for authenticating against the API, and specifically how we can perform unattended authentication from PowerShell for use with automated processes such as Azure Automation runbooks.  
We also had a look at current automation capabilities against the Microsoft Intune service, and how we can leverage the Microsoft Graph Intune API for Intune automation using PowerShell.

**Resources**

- [Working with Intune in Microsoft Graph](https://developer.microsoft.com/en-us/graph/docs/api-reference/beta/resources/intune_graph_overview)  
    **Important note from the article (per October 2017):** *Intune APIs under the /beta version in Microsoft Graph are in preview and are subject to change. Use of these APIs in production applications is not supported.*
- [Integrating applications with Azure Active Directory](https://docs.microsoft.com/nb-no/azure/active-directory/develop/active-directory-integrating-applications)
- [Using PowerShell and OAuth](https://foxdeploy.com/2015/11/02/using-powershell-and-oauth/)
- [Azure Active Directory Authentication Library (ADAL)](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-libraries)
- [Understanding user and admin consent](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-devhowto-multi-tenant-overview#understanding-user-and-admin-consent)