---
layout: post
title:  "Announcing the Data Protection Manager Community Extensions PowerShell module"
date:   2017-07-03 07:47:20 +0200
comments: true
categories: Backup
---

In this article, we are going to introduce a new PowerShell module called Data Protection Manager Community Extensions. Before we get started, let us have a quick overview of what products the module support.

**Background**

System Center Data Protection Manager (DPM) is an enterprise backup system which provides functionality to back up data from a source location to a target location connected to the DPM server. Data with a short amount of retention time is typically stored on disks attached to the DPM server, while data with longer retention times (up to 99 years) traditionally has been stored to a tape library. Online backup to Azure for long term retention was introduced in DPM 2012 SP1. There were some limitations to the initial offering, but the current version supports all workloads supported by DPM including files, folders, volumes, system state, Bare Metal Recovery, Exchange, SQL Server, Sharepoint and virtual machines running on both Hyper-V and VMware.

![alt](/images/azure-backup-overview.png){:height="528px" width="600px"}

Image credit: [Microsoft](https://azure.microsoft.com/en-us/documentation/articles/backup-introduction-to-azure-backup/)

Traditionally, customers would need to buy licenses for the System Center suite in order to use DPM. At the end of 2015, Microsoft [released](https://azure.microsoft.com/en-us/blog/announcing-microsoft-azure-backup-server/) what is called Microsoft Azure Backup Server. This is a modified and rebranded version of DPM where tape support and System Center integration is removed. System Center licenses is also not required, instead an active Azure subscription with a valid Azure Backup vault is required.
At the end of May 2017, Microsoft [released](https://azure.microsoft.com/en-us/blog/protect-windows-server-2016-and-vcenter-esxi-6-5-using-azure-backup-server/) V2 of Microsoft Azure Backup Server, adding support for Windows Server 2016 and new features such as [Modern Backup Storage](https://azure.microsoft.com/en-us/blog/introducing-modern-backup-storage-with-azure-backup-server-on-windows-server-2016/).

**Introducing Data Protection Manager Community Extensions**

DPM provides both a Graphical User Interface (GUI) and a [PowerShell module](https://docs.microsoft.com/en-us/powershell/systemcenter/systemcenter2016/dataprotectionmanager/vlatest/DataProtectionManager) for management.
The PowerShell module is available as a shortcut on the desktop and Start menu/screen by default:

![alt](/images/DPM_icon.png)

However, all this shortcut does is starting an instance of powershell.exe which invokes the DpmCliInitScript.ps1 initialization script from the program files bin directory. Another option is to manually import the module from a regular PowerShell session:

![alt](/images/DPM_CX_01.png){:height="116px" width="600px"}

Alternatively, you can rely on module auto loading if you are running Windows PowerShell 3.0 or later. When doing so, the DPM module will be loaded automatically when a DPM cmdlet such as Get-DPMProtectionGroup is run. You might wonder why there is a large number of aliases in the DataProtectionManager module. The reason is that DPM did not follow the best practice of using a product specific prefix in the first versions of the product. When that practice was implemented, a number of aliases was created for the old cmdlet names in order to not break existing scripts.
After loading the module, we can run Connect-DPMServer to connect to an instance of DPM Server. If that runs successfully we can run other DPM cmdlets such as Get-DPMProtectionGroup:

![alt](/images/DPM_CX_02.png){:height="150px" width="600px"}

One “quirk” about the DPM module is that it is very object oriented. That`s fundamentally a good thing, but the implementation in this module could have been more user friendly. An example is working with recovery points (a point in time backup of a specific data source such as a SQL database). The -Datasource parameter of Get-DPMRecoveryPoint does not accept a string as input, the user will need to pass in an object:

![alt](/images/DPM_CX_03.png)

A more concrete example is something which should be a simple task for a DPM administrator: Retrieve all recovery points for a specific data source using PowerShell. In an ideal implementation, the administrator could simply type Get-DPMRecoveryPoint -ProtectionGroup 'SQL - 3 days' -DataSource VirtualManagerDB. However, the following technique is needed:

![alt](/images/DPM_CX_04.png){:height="49px" width="600px"}

![alt](/images/DPM_CX_14.png){:height="85px" width="600px"}

Notice that we needed to retrieve two objects before we had what we needed for the -Datasource parameter of Get-DPMRecoveryPoint. Also note that Get-DPMDatasource and Get-DPMProtectionGroup does not accept string input for selecting a specific item, we had to use Where-Object to get the specific items we wanted.
Having control over the latest recovery points is a crucial task for every DPM administrator, so having to go through a rather difficult process to get such basic information in PowerShell is very inconvenient.
One of the goals of the project I have started is to ease common tasks such as this example. Let me introduce the *Data Protection Manager Community Extensions PowerShell module*:  
It contains commands for working with both DPM and Microsoft Azure Backup Server. The goal of this module is to make it easier to manage the two products by providing functionality not included in the built-in Data Protection Manager PowerShell module (also known as DPM Management Shell/Microsoft Azure Backup Management Shell).

The initial version of the module contains the following functions:
- **Get-DPMCXAgent** - Returns information about a DPM agent, including version information and attached DPM servers.
- **Get-DPMCXAgentOwner** - Returns information about which DPM servers a DPM agent is attached to and what data sources is protected.
- **Get-DPMCXRecoveryPointStatus** - Returns information about recovery points older than a specified value, for example 2 days.
- **Get-DPMCXServerConfiguration** - Returns information about the DPM server configuration, including properties relevant for best practices such as page file size and SQL Server memory configuration.
- **Get-DPMCXVersion** - Helper function used by other commands which needs to translate a build number to a friendly name, such as returning 'Update Rollup 11 for System Center 2012 R2 Data Protection Manager' as the friendly name for build number 4.2.1552.0. Specify -ListVersion to return all build numbers for System Center DPM 2012 R2, System Center DPM 2016 and Microsoft Azure Backup Server. 
- **Get-DPMCXSizingBaseline** - Returns basic sizeline guidelines for DPM 2012 - 2012 R2.
- **Get-DPMCXMARSAgent** - Returns information about a Microsoft Azure Recovery Services Agent agent, including version information.
- **Get-DPMCXMARSVersion** - Helper function used by other commands which needs to translate a build number to a friendly name, such as returning 'Microsoft Azure Recovery Services Agent: May 2017' as the friendly name for build number 2.0.9077.0. Specify -ListVersion to return all build numbers. 
- **New-DPMCXRecoveryPointStatusReport** - Builds an HTML report for the output generated by Get-DPMRecoveryPointStatus which can be saved to a file path or sent to an e-mail address.
- **New-DPMCXServerConfigurationReport** - Builds an HTML report for the output generated by Get-DPMServerConfigurationReport which can be saved to a file path or sent to an e-mail address.
- **New-DPMCXMARSAgentReport** - Builds an HTML report for the output generated by Get-DPMCXMARSAgent which can be saved to a file path or sent to an e-mail address.
- **Test-DPMCXComputer** - Returns information about whether DPM is installed on a computer. The output included properties such as IsInstalled,IsDPMServer and version information.

The module is published to the PowerShell Gallery, which means you can install it using the following command from the PowerShellGet module:

*Install-Module -Name DataProtectionManagerCX*

If you do not have the option to leverage the PowerShellGet module, manual installation instructions as well as information about requirements is available in the [GitHub repository](https://github.com/janegilring/DataProtectionManagerCX) for the module.

After installation, you can leverage Get-Command to list available functions:

![alt](/images/DPM_CX_05.png){:height="130px" width="600px"}

Below is some sample output from some of the functions.
Get-DPMCXVersion with the -ListVersion parameter can be used to get an overview of the latest versions of DPM and Microsoft Azure Backup Server.

![alt](/images/DPM_CX_06.png){:height="164px" width="600px"}

The IsInstalled and IsDPMServer properties can be leveraged in inventory scripts, for example:

![alt](/images/DPM_CX_07.png){:height="126px" width="600px"}

![alt](/images/DPM_CX_08.png){:height="129px" width="600px"}

Get-DPMCXAgentOwner parses the files in the agents ActiveOwner directory and returns the assosicated DPM server along with the filename which indicates the name of the protected data source:

![alt](/images/DPM_CX_09.png){:height="87px" width="600px"}
 
Get-DPMCXRecoveryPointStatus returns information about the latest recovery point older than a specified value for each protected data source.

![alt](/images/DPM_CX_10.png){:height="276px" width="600px"}

Note that most functions in this module should return verbose information when -Verbose is added to give more information about what it`s doing. Here is an example from the previous function:

![alt](/images/DPM_CX_11.png){:height="108px" width="600px"}

New-DPMCXRecoveryPointStatusReport does not generate any output, since it builds an HTML report for the output generated by Get-DPMRecoveryPointStatus and sends it to an e-mail address in the example below:

![alt](/images/DPM_CX_12.png){:height="21px" width="600px"}

Instead of reinventing the wheel with regards to generating reports, the PScribo module is leveraged by New-DPMCXRecoveryPointStatusReport to generate the HTML report.
Here is an example of how it looks like:

![alt](/images/DPM_CX_13.png){:height="221px" width="600px"}

In this example, the recovery points older than 1 day is marked with a red background.
It is also easy to combine the commands in the DataProtectionManagerCX module with other modules. One example is getting computer names from Active Directory and leverage the DataProtectionManagerCX module to gather information about which computers has DPM installed:

{% gist 17bea68b3946f2542e7fbfc7da58c707 %}

Going forward, more functionality is planned. There is a lot of existing DPM scripts out there, for example on the TechNet Gallery. Any owners of existing scripts are welcome to add their scripts to the DataProtectionManagerCX module. If you are not familiar with the process of forking a repository, making changes (adding a function for example) and submitting a pull request, you can simply submit an issue where you provide some information about the script and we will assist with the process of adding it to the module (preferably as a function).
Other examples of planned functionality is adding the script for estimating Azure Backup billing usage for DPM datasources in [this article](https://blogs.technet.microsoft.com/dpm/2015/03/31/explaining-the-new-azure-backup-pricing-for-dpm-and-ea-customers/) as a function.
Any other suggestions are also welcome, feel free to submit an [issue](https://github.com/janegilring/DataProtectionManagerCX/issues) on the GitHub repository or leave a comment on this article (for problems and bugs I would prefer that issues is logged to the GitHub repository).

**Resources**

Microsoft Azure Backup Server – [download](https://www.microsoft.com/en-us/download/details.aspx?id=55269)
Microsoft Azure Backup Server – [documentation](https://docs.microsoft.com/en-us/azure/backup/backup-azure-microsoft-azure-backup)
Data Protection Manager (DPM) - [documentation](https://docs.microsoft.com/en-us/system-center/dpm/)

**Summary**

In this article we have looked briefly at System Center Data Protection Manager and Azure Backup Server. We also looked at the background for creating the Data Protection Manager Community Extensions PowerShell module, such as the fact that the native DPM PowerShell module is very object oriented and in many cases not very easy to use for common tasks. The goal of the Community Extensions module is to make difficult tasks easier, as well as add additional capabilities such as scheduled e-mail reports.