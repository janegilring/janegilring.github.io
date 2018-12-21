---
layout: post
title:  "Repair Active Directory computer account secure channel"
date:   2017-02-08 18:31:40 +0200
comments: true
categories: ActiveDirectory
---

# Challenge

Just like user accounts, computer accounts in Active Directory also has passwords that the computers  
 use to authenticate to the domain controllers in the domain. A difference is that we never see the  
 password for the computer account, as this is handled automatically by the system. This is a very  
 robust functionality, like what is being used for a [Managed](https://technet.microsoft.com/en-us/library/dd560633(v=ws.10).aspx) or [Group Managed Service Accounts](https://technet.microsoft.com/en-us/library/hh831782(v=ws.11).aspx).  
There are some scenarios which might cause the password stored on the domain controllers to get out  
of sync with the password stored locally on a computer. One of the most common reasons for this is  
 someone joining a computer with the same name to the domain. The new computer will take ownership  
 over the machine account and reset the stored password, leading to the existing computer not being  
 able to authenticate to the domain. Another common scenario is checkpoints in virtual machines.  
 If a virtual machine is reverted to an earlier point in time, the machine password might have  
 changed since the checkpoint was taken and thus break the secure channel against the Active  
 Directory domain. 

# Solution

Let us have a look at one of these scenarios in a demo, and see how to resolve it:  
In this demo, I have a domain-joined Windows computer where I have opened Windows PowerShell with   
administrative privileges. To verify that the password stored locally is in sync with the domain  
controllers (also referred to as a “secure channel”), we can use the [Test-ComputerSecureChannel](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.management/test-computersecurechannel)  
cmdlet:  

![alt](/images/Test-ComputerSecureChannel_01.png)

You may also add the -Verbose switch parameter to get additional information:

![alt](/images/Test-ComputerSecureChannel_02.png)

Let`s assume that someone have added a computer with the same name to the domain. When trying to  
logon to the existing computer using a domain account, we will receive the error *"The trust 
relationship between this workstation and the primary domain failed”*:  

![alt](/images/Test-ComputerSecureChannel_03.png)

Let`s re-run the Test-ComputerSecureChannel command:  

![alt](/images/Test-ComputerSecureChannel_04.png)

As you might have suspected when you receive the error “The trust relationship between this  
workstation and the primary domain failed”, the secure channel is broken.  
If adding the other computer to the domain with was a mistake, and we want to bring ownership of  
the computer account in Active Directory back to the existing computer, we can use the -Repair  
switch parameter for Test-ComputerSecureChannel:  

![alt](/images/Test-ComputerSecureChannel_05.png)

As you can see, we also need to specify credentials for a domain account with the appropriate 
permissions to perform the operation. After running the command we can see that the secure channel 
is healthy:

![alt](/images/Test-ComputerSecureChannel_06.png)

There is no need to reboot the machine, you can log on using a domain account immediately.
You should also see that the PasswordLastSet property for the computer object in Active Directory  
 is updated:

![alt](/images/Test-ComputerSecureChannel_07.png)

An alternative to using Test-ComputerSecureChannel -Repair to reset the password for a computer  
account is the [Reset-ComputerMachinePassword](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.management/reset-computermachinepassword) command:

![alt](/images/Test-ComputerSecureChannel_08.png)

It will perform the same task, and might be easier to remember. After running the command we can  
see that the PasswordLastSet property is updated again in Active Directory:

![alt](/images/Test-ComputerSecureChannel_09.png)

## Other root causes

*This section was added 2018-12-21*

Sometimes the repair operation is not successful, which might be caused by other underlying root causes.

One example is port exhaustion. I once ran into a situation where Test-ComputerSecureChannel returned false and  Test-ComputerSecureChannel -Repair failed with the error *The network path was not found.*

The following was run to unjoin the machine from the domain:

```powershell
$LocalAdminCred = Get-Credential
Remove-Computer -WorkgroupName TEMP -UnjoinDomainCredential $LocalAdminCred
```

Since the machine was running SQL Server and serving active connections we did not want to reboot, hence an immediate domain join was initiated:

```powershell
$DomainCred = Get-Credential
Add-Computer -Credential $DomainCred -DomainName powershell.no
```

This failed with the following error:
*Add-Computer : Computer 'SRV01' failed to join domain 'powershell.no' from its current workgroup 'TEMP' with following error message: The network path was not found.
At line:1 char:1*

After testing basics such as DNS, we noticed only FQDN lookups worked. Hence we added the domain name as a suffix on the network adapter.
After that DNS queries worked as expected, and we re-tried the domain join:

*Add-Computer : Computer 'SRV01' failed to join domain 'powershell.no' from its current workgroup 'TEMP' with following error message: The name limit for the local computer network adapter card was exceeded.
At line:1 char:1*

Accessing file shares such as \\server\share also failed with the same error.

After doing some research on the error message, we noticed a huge output of netstat -a/Get-NetTCPConnection.

netstat -ab revealed the process name which exhausted the range of dynamic ports.

After some research after this incident it turns out it is also possible to get this information - including the username for the owning process - using PowerShell ([credit](https://stackoverflow.com/questions/44509183/powershell-get-nettcpconnection-script-that-also-shows-username-process-name)):

```powershell
Get-NetTCPConnection | 
Select LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess , @{n="ProcessName";e={(Get-Process -Id $_.OwningProcess).ProcessName}} , @{n="UserName";e={(Get-Process -Id $_.OwningProcess -IncludeUserName).UserName}}| 
Where {$_.State -eq"Established"} | Group-Object ProcessName -NoElement | Sort-Object -Property Count -Descending
```

After killing the problematic process the machine could successfully be re-joined to the domain - without any need for a reboot.

# Summary

Test-ComputerSecureChannel was introduced in PowerShell 2.0 (built-in to Windows 7/Server 2008 R2)   
while Reset-ComputerMachinePassword was introduced in PowerShell 3.0 (built-in to Windows 8/Server  
2012). Prior to the introduction of these cmdlets we could use  
netdom resetpwd /s:server /ud:domain\User /pd:* to reset a machine password and  
nltest.exe /sc_verify:domain.local to verify the secure channel. Obviously, the syntax and  
discoverability of the PowerShell alternatives is much better and should be the preferred options.  