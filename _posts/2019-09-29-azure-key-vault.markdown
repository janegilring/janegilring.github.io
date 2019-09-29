---
layout: post
title:  "Using Azure Key Vault for local administrator password rotation"
date:   2019-09-29 17:30:00 +0200
comments: true
categories: PowerShell, Azure
---

In this article, we are going to see how we can leverage Azure Key Vault for storing local administrator passwords configured on Windows Servers. Before we get started, let us have a quick overview of what Azure Key Vault is.

## Azure Key Vault

*Azure Key Vault is a tool for securely storing and accessing secrets. A secret is anything that you want to tightly control access to, such as API keys, passwords, or certificates. A vault is logical group of secrets.*

*Anybody with an Azure subscription can create and use key vaults. Although Key Vault benefits developers and security administrators, it can be implemented and managed by an organization’s administrator who manages other Azure services. For example, this administrator can sign in with an Azure subscription, create a vault for the organization in which to store keys, and then be responsible for operational tasks like these:*

- *Create or import a key or secret*
- *Revoke or delete a key or secret*
- *Authorize users or applications to access the key vault, so they can then manage or use its keys and secrets*

![alt](/images/azure-key-vault.png)

Source, and more information: [https://docs.microsoft.com/en-us/azure/key-vault/basic-concepts](https://docs.microsoft.com/en-us/azure/key-vault/basic-concepts).

Usage is not limited to applications, as we can easily create and retrieve secrets using the Azure CLI, Azure PowerShell module or any other tool which can interact via a REST interface.

## Scenario

Best practice when it comes to security involves rotating secrets on a regular basis.
One such example is Active Directory domain joined computers.

Microsoft provides a product for this scenario:

*The ["Local Administrator Password Solution"](https://www.microsoft.com/en-us/download/details.aspx?id=46899) (LAPS) provides management of local account passwords of domain joined computers. Passwords are stored in Active Directory (AD) and protected by ACL, so only eligible users can read it or request its reset.*

While this works great for domain joined computers, we will look at an alternate approach which is not limited for use with Active Directory. Although that is the provided example, it can easily be extended to include other environments - such as root users for Linux machines, for example. Another scenario might be many Active Directory forests consolidating their local password management in Azure Key Vault.

## Solution

The solution consist of several building blocks:

- An Azure Key Vault
- A hybrid runbook worker
- An Azure Automation runbook

An Azure Key Vault dedicated for storing secrets will need to be provisioned. In this example, a single Key Vault is used to store the local administrator password for all Windows Servers in an Active Directory domain. Access to this vault needs to be restricted, similarly to how access to the Domain Admins security group in Active Directory is restricted to only a limited set of people. The only use case for retrieving passwords from this vault should be when the domain trust is broken and we need to log on to a server with a local administrator account. One way to grant just-in-time access to this vault could be securing it with [Azure Active Directory Privileged Identity Management](https://docs.microsoft.com/en-us/azure/active-directory/privileged-identity-management/pim-configure), where approvals can be leveraged before granting access to the vault.

Next, we will need access to all target servers - in this example all servers within the Active Directory domain - except Domain Controllers, which does not have any local accounts.
In this example, we will leverage Azure Automation to accomplish this. We will then need an [Azure Automation Hybrid Runbook Worker](https://docs.microsoft.com/en-us/azure/automation/automation-hybrid-runbook-worker) where our PowerShell runbook can execute from.

![alt](/images/azure-automation-hybrid-worker.png)

The final piece is the runbook which will contain the actual code for performing password rotation as well as storing them in the Key Vault.

I have written an example runbook called [Reset-LocalAdministratorPassword.ps1](
https://github.com/janegilring/PSCommunity/blob/master/Azure/Automation/Runbooks/Reset-LocalAdministratorPassword.ps1).

It accepts 4 parameters:

- PasswordLength
- SpecialCharCount
- PasswordAgeThreshold
- Force

The runbook logic is the following:

- Retrieve all computer accounts from Active Directory with "Server" present in the operating system property, except domain controllers
- Connect to Azure Key Vault
- Foreach Server in Servers
  - Check if existing entry for the server is present in Key Vault
    - If yes
      - Check when password was last updated
    - If older than specified threshold (default 180 days): Update password and store in vault
      - Check if Force parameter is true
        - If yes: Update password
    - If no: Create password and store in vault

The runbook would need to be scheduled within Azure Automation to run on a regular basis, for example once every day.

Example output from the Azure Automation runbook:

![alt](/images/azure-automation-password-runbook.png)

When re-run after the initial execution, no changes are made since the password threshold is not met yet:
![alt](/images/azure-automation-password-runbook-02.png)

Example Azure Key Vault content:

![alt](/images/azure-key-vault-example.png)

![alt](/images/azure-key-vault-example-02.png){:height="481px" width="400px"}

You might notice that there was no error logging configured in the runbook. The reason for that is we can simply [Forward job status and job streams from Automation to Azure Monitor logs](https://docs.microsoft.com/en-us/azure/automation/automation-manage-send-joblogs-log-analytics). After doing so, we can create alert rules in Azure Monitor to notify an action group responsible for monitoring when errors occurs in the runbook - either by SMS, e-mail, a Logic App sending notifications to Slack or Microsoft Teams and so on.

An example log query to monitor the error stream in our example runbook:
**AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and StreamType_s == "Error"  |
where RunbookName_s == "Reset-LocalAdministratorPassword"**

## Summary

In this article we have looked at the challenge of rotating passwords on Windows Servers on a regular basis, using a PowerShell runbook executing on a hybrid runbook worker to perform the actual work.
Azure Automation works great for hybrid scenarios like this. Another option when working with more cloud native services might be executing the code from Azure Functions, eliminating the need for a server to run the code on. This is something we will look at for a different scenario in a future article.
