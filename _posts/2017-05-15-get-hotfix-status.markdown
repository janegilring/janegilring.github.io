---
layout: post
title:  "Using PowerShell to test whether hotfixes is installed"
date:   2017-05-15 12:31:40 +0200
comments: true
categories: Inventory
---

**Challenge**

In certain situations, there is a need to quickly determine whether a hotfix is deployed to a set of computers.  
A good example of this is the WannaCrypt attacks which occured in May 2017.  

Microsoft published a [Customer Guidance for WannaCrypt attacks](https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/) with references to which hotfixes  
is applicable for protecting against that specific vulnerability.

Given that there are updates for many different versions of Windows, there is a lot of  
hotfixes to look for and this might be a cumbersome task if there isn't any good inventory system available for giving a quick overview of the current state.


**Solution**

PowerShell 2.0 introduced the [Get-HotFix](https://msdn.microsoft.com/en-us/powershell/reference/5.0/microsoft.powershell.management/get-hotfix) cmdlet  
for checking hotfixes that have been applied to a system.

To make it more convenient to leverage Get-HotFix, I've written a function called Get-HotFixStatus  
which will connect to target computers using PowerShell Remoting. One of the benefits is that you  
do not have to rely on WMI being enabled in the firewall, which is necessary if you use -ComputerName  
parameter of Get-HotFix. Of course, it does require you to have PowerShell Remoting (TCP 5985/5986 by default) enabled. However, it is pretty uncommon to not have PowerShell Remoting these days (it`s enabled by default since Server 2012).

Get-HotFixStatus lets you specify one or more Knowledge Base (KB) IDs to the -Id parameter, but you 
 can also pass in an array of KB IDs. This is useful if you have a list of all hotfixes for a specific 
  vulnerability, such as the one in the mentioned WannaCrypt attacks.

It also supports getting computers directly from Active Directory by using the -OUPath and  
-InactiveADComputerObjectThresholdInDays parameters.
By default it will query Active Directory for computer accounts starting with "Windows Server*" in  
the operating system attribute, in addition to excluding Cluster Name Objects (CNOs). If you want  
to override this behaviour, you can specify the -LdapFilter parameter to pass in your own filter.

If you have a company internal "tools module", you could simply just add the function to that.
If you want to leverage it standalone, you can simply just download and "dot source" it.

Here is some examples:
{% gist 80b683684d84ed4f3dc4641d26342271 %}

You can find the same examples in the command`s help examples:
![alt](/images/2017-05_15_hotfix-status_01.png){:height="436px" width="600px"}

Example output:
![alt](/images/2017-05_15_hotfix-status_03.png){:height="218px" width="480px"}


Sample Excel-report from my lab enviroment:
![alt](/images/2017-05_15_hotfix-status_02.png){:height="188px" width="350px"}

The function is available in my [PSCommunity module on GitHub](https://github.com/janegilring/PSCommunity/blob/master/Inventory/Get-HotfixStatus.ps1), feel free to log issues or suggest new features in the repository.

**Summary**

PowerShell makes it easy to generate inventory reports of installed hotfixes by using built-in  
functionality such as Get-HotFix and PowerShell Remoting. It`s also easy to leverage community  
resources such as the ImportExcel module for exporting the data to a consumable format.