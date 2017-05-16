---
layout: post
title:  "Using PowerShell to disable or remove SMB1"
date:   2017-05-16 15:48:40 +0200
comments: true
categories: Security
---

**Background**

Following the previous article about [Using PowerShell to test whether hotfixes is installed](http://www.powershell.no/inventory/2017/05/15/get-hotfix-status.html),  
another challenge these days is to lower the attack surface as much as possible.  
There are many ways to do this, such as restricting what firewall ports is open, having a good  
systems in place for patch management, and so on.
One mitigation related to the [WannaCrypt attacks](https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/)  
 which is a relevant topic these days, is to disable the SMB 1 protocol on as many systems as possible.

Version 1 of the protocol is only needed by operating systems which is no longer supported by Microsoft:
-  SMB 1.0 – Windows 2000, Windows XP, Windows Server 2003 and Windows Server 2003 R2
-  SMB 2.0 – Windows Vista and Windows Server 2008
-  SMB 2.1 – Windows 7 and Windows Server 2008 R2
-  SMB 3.0 – Windows 8 and Windows Server 2012
-  SMB 3.02 – Windows 8.1 and Windows Server 2012 R2
-  SMB 3.1.1 – Windows 10 and Windows Server 2016

On all other systems, it is a good idea to consider either disabling or removing the SMB 1.0 protocol.  

Ned Pyle, who is the owner of the SMB protocol in Microsoft, has written a great [article](https://blogs.technet.microsoft.com/filecab/2016/09/16/stop-using-smb1/) about why this is a good idea.

Quote from the article:  
*The original SMB1 protocol is nearly 30 years old, and like much of the software made in the 80’s, it was designed for a world that no longer exists*

Microsoft also have a great article on how to do this:
[How to enable and disable SMBv1, SMBv2, and SMBv3 in Windows and Windows Server](https://support.microsoft.com/en-us/help/2696547/how-to-enable-and-disable-smbv1,-smbv2,-and-smbv3-in-windows-vista,-windows-server-2008,-windows-7,-windows-server-2008-r2,-windows-8,-and-windows-server-2012)



**Examples on how to remove SMB 1 using PowerShell**

There are some excellent examples in the mentioned article on how to disabling and removing SMB 1 using PowerShell, but I wanted to show some additional techniques on how to remove it.
Before showing that I would like to highlight the recommended steps to go through first:

-  Step 1 - Identify what systems in your organization you can safely remove SMB1 from
-  Step 2 - In a test environment, verify that removing or disabling SMB1 does not have any impact on applications or services in use
-  Step 3 - Remove or disable SMB 1

Here is some examples on how (and where) to remove SMB 1 using PowerShell:  

{% gist 4a3f0731daa78fdb791be05af2bbfd14 %}

