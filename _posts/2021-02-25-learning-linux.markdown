---
layout: post
title:  "Getting into the Linux world as a PowerShell user"
date:   2021-02-25 09:00:00 +0200
comments: true
categories: Linux, PowerShell, DevOps
---

# Introduction

Up until the past couple of years or so I have mainly been working with Windows-based systems. However, as mentioned in a previous post - when technology evolves there is a need to adapt - and getting into the Linux side of things is one key area given the frequent use of Linux in modern technologies such as containers.

Since starting working more and more with containers and Kubernetes, I have been exposed more and more to Linux and learned some fundamentals such as looking into logs and managing services. As time passed I found a need to dig a little deeper and learn the basics on Linux system administration, as these skills is useful both when working inside containers but also in general as we see more and more greenfield deployments being based on Linux server distributions rather than Windows Server.

I think one advantage before starting is that I have been working a lot with PowerShell since it was released as Windows PowerShell 1.0 back in 2006 - which is both a command-line administration shell, a scripting language and an automation engine. It was a very important tool for any IT Pro managing Windows-based systems (both desktops and servers) in the following decade. Then, in August of 2016, Microsoft announced it`s plans to make PowerShell open source and cross-platform - available on Linux and macOS in addition to Windows. Since PowerShell is built on top of the .NET Framework, the decision of that team to make .NET cross-platform also layed the foundation for PowerShell to do the same thing. Generally availability of PowerShell Core 6.0 was announced in January 2018. In the initial version "Core" was added to the name to distinguish it from Windows PowerShell as well as highlight that it was running on the .NET Core version of .NET Framework. However, that lead to some confusion for the user base, as one might get the impression that "Core" meant a stripped down or light version of PowerShell. In fact, it was lacking some APIs and functionality, since porting features to a brand new framework took some time. However, the .NET team changed their branding from .NET Core 3.1 to .NET 5 in the same timeframe -  as they thought feature and compatibility with the older/Windows-only version was sufficient. Hence, from PowerShell 7, which was released in March 2020 - the "Core" part of the branding was removed for PowerShell as well.

# Learning Linux Systems Administration

I have some experience with PowerShell on Linux, deploying it with SSH-remoting enabled at a customer. This enables cross-platform remoting and is very cool from a technology perspective. Here is one example:

![alt](/images/2021-02-25-ps-remoting.gif)

Knowing PowerShell - which is object-oriented, much like the Windows operating system is - is fundamentally very different from Linux - which is largely based on defining configurations in text files. Even system information such as open handles by a process is exposed as text-files using a pseudo file-system.

Hence, it doesn`t help much knowing PowerShell in this scenario, without also some foundational skills for managing the operating system.
Combining the powerful capabilities of PowerShell with the basic understanding of managing Linux on the other hand, is a great combination in my opinion.

I started out by setting a goal: I wanted to learn the skills required to pass the Linux Foundation Certified Systems Administrator exam in order to get foundational knowledge of the Linux operating system.

In the beginning, I must admit I felt a bit lost without my three faithful PowerShell commands:
- `Get-Command` - The Get-Command cmdlet gets all commands that are installed on the computer, including cmdlets, aliases, functions, filters, scripts and applications (which means binaries in the PATH, such as ipconfig and such). Example: - Get-Command *disk*
- `Get-Help` - Displays information about PowerShell commands and concepts. Examples: Get-Help about_try_catch and Get-Help New-AzVm.
- `Get-Member` - The Get-Member cmdlet gets the members, the properties and methods, of objects. PowerShell has a rich formatting system, which means only specific properties will be displayed by default (it depends on things like the number of properties and whether a formatting defintion has been made for the specific type of object). By using Get-Member (and sometimes Format-List * to see actual property values) we can discover how an object looks like and pick the information we want by pipine to Select-Object.

I did know about the man command, which can display the manual page for a specified command. However, that does not help much if you do not know the name of the command. Later on, I discovered the apropos command - which in reality is an alias for man -k. This makes it possible to specify a keyword to search through all of the man pages. For example, if one wants to format a partition on a disk, but can`t remember the command one can search for - try: apropos disk. This will return tools like fdisk, gdisk and so on - and you will likely find the tool you want. This helped tremendously for me.

> Give a man a fish and you feed him for a day; teach a man to fish and you feed him for a lifetime. --Mainmonides

I think this quote fits very well in this scenario: Get to know the base tools to discover what you need and you can find what you need when you need it.

Another "quirk" for someone used to work in PowerShell where commands is built on an intuitive syntax based on the Verb-Noun syntax - for example Get-Service, Start-Service and Stop-Service - is the rather non-intuitive nature of command names in Linux.

For example, the command name awk - which is an incredible powerful command designed for text processing and typically used as a data extraction and reporting tool - and almost a complete language on it`s own - does not describe what the tool does by any means. The name is actually composed by the first letter in the lastname of it's creators (Alfred Aho, Peter J. Weinberger, Brian Kernighan). Sed is a different example, a utility that parses and transforms text. Its name is short for "stream editor" - which is a little better, but still is hard to guess based on the name alone. The rationale for short names is likely to make it shorter to type for users, but that is largely solved by the concept of aliases in my opinion. For example in PowerShell, there are some really long command (cmdlet) names (such as Enable-AzOperationalInsightsLinuxCustomLogCollection, which is 52 characters long and thus very verbose), but aliases makes it more convenient to use them when working interactively. In scripts, the best practice is to use the full command names for readability and clarity.

Intellisense and tab-completion is other helpful features in this space.
For example, I have gotten accustomed to use the Linux alias ls instead of Get-ChildItem - which is the PowerShell command to list items in a directory. Even though I might have used the other default alias dir, I tend to choose the shortest ones for effiency. On the topic of efficiency, check out [Predictive Intellisense](https://devblogs.microsoft.com/powershell/announcing-psreadline-2-1-with-predictive-intellisense/):

*Predictive IntelliSense is an addition to the concept of tab completion that assists the user in successfully completing commands. The prediction suggestion appears as colored text following the user’s cursor. This enables new and experienced users of PowerShell to discover, edit, and execute full commands based on matching predictions from the user’s history and additional domain specific plugins.*

![alt](/images/2021-02-25-intellisense.jpg)
*Image credit: Microsoft*

In this example, the gray text after the cursor is Predictive text based on either the history or a plugin (such as the Az.Predictor for the Azure module). By default one can use RightArrow in order to use the Predictive Intellisense.

Here is a practical example where I first start typing out the Enter-PSSession command and then change to typing the name of a computer I want to connect to:

![alt](/images/2021-02-25-intellisense-demo.gif)

Getting all of the command history where the text has been used before is very conventient.

See the blog post for additional details on how to install and configure Predictive Intellisense. I have used this feature heavily since it was released November 10th 2020, and it has saved me significants amount of time - especially for long commands I use frequently.

As noted in the article, Predictive Intellisense works on Windows PowerShell 5.1 and PowerShell 7.0+, meaning it also works on Linux and macOS. Hence, I am also using this everywhere I use PowerShell - such as Azure Cloud Shell and WSL2 (Windows Subsystem for Linux).

## Crescendo
Another exciting new module is [PowerShell Crescendo](https://devblogs.microsoft.com/powershell/announcing-powershell-crescendo-preview-1/), a framework to rapidly develop PowerShell cmdlets for native commands, regardless of platform. Crescendo provides the tools to easily wrap a native command to gain the benefits of PowerShell cmdlets (such as object orientation). As noted in the article, this means that the output objects allow you to take advantage of all the post processing tools such as Sort-Object and Where-Object.
An example provided in the article on the PowerShell team blog is a wrapper around the Linux apt command:
```powershell
Get-InstalledPackage | Where-Object { $_.name -match "libc"}
```

As we can see, as a PowerShell user one can make use of existing knowledge for filtering output. I think this is a perfect example of what Jeffrey Snover (Microsoft Technical Fellow and "father" of PowerShell) and the PowerShell has been aiming for since the beginning:

![alt](/images/2021-02-25-snover-quote.png)

Although this is great, we can`t assume that PowerShell is available on every Linux system we get manage - hence I believe it is still needed to know some basics of the native tooling. Such as text processing utilities like grep, sed and awk. Another example is container images. They are often stripped down as much as possible to minimize footprint - even tools we take for granted such as ping or ip might be excluded. Hence, we need to use the tools available if doing some interactive debugging in such a  scenario.

## History

Another fascinating aspect of learning Linux is all of the history in the operating system. An example is the archiving utility tar. Although its name is short for tape archiver, it is still used today where tape systems is almost non-existent. Now it is used to archive and extract archives to and from any media - as long as it have a supported file system.

Another example is the previously mentioned sed utility. According to [Wikipedia](https://en.wikipedia.org/wiki/Sed), it was was developed from 1973 to 1974 by Lee E. McMahon of Bell Labs, and is available today for most operating systems. By the time of this writing, that is almost 50 years later. I find this very interesting, that a built-in tool is still actively developed and highly used so long after it was initially developed. In the fast-paced world of computing, 50 years is like centuries!


# LFCS: Linux Foundation Certified Systems Administrator

I started out studying for this exam a some weeks before the scheduled exam date. I bought the Course + Exam bundle, and got the course [Essentials of Linux System Administration (LFS201)](https://training.linuxfoundation.org/training/essentials-of-linux-system-administration/) in addition to the exam:

- Chapter 1. Course Introduction
- Chapter 2. Linux Filesystem Tree Layout
- Chapter 3. Processes
- Chapter 4. Signals
- Chapter 5. Package Management Systems
- Chapter 6. RPM
- Chapter 7. DPKG
- Chapter 8. yum
- Chapter 9. zypper
- Chapter 10. APT
- Chapter 11. System Monitoring
- Chapter 12. Process Monitoring
- Chapter 13. Memory: Monitoring Usage and Tuning
- Chapter 14. I/O Monitoring and Tuning
- Chapter 15. I/O Scheduling
- Chapter 16. Linux Filesystems and the VFS
- Chapter 17. Disk Partitioning
- Chapter 18. Filesystem Features: Attributes, Creating, Checking, Mounting
- Chapter 19. Filesystem Features: Swap, Quotas, Usage
- Chapter 20. Th ext2/ext3/ext4 Filesystems
- Chapter 21. The XFS and btrfs Filesystems
- Chapter 22. Encrypting Disks
- Chapter 23. Logical Volume Management (LVM)
- Chapter 24. RAID
- Chapter 25. Kernel Services and Configuration
- Chapter 26. Kernel Modules
- Chapter 27. Devices and udev
- Chapter 28. Virtualization Overview
- Chapter 29. Containers Overview
- Chapter 30. User Account Management
- Chapter 31. Group Management
- Chapter 32. File Permissions and Ownership
- Chapter 33. Pluggable Authentication Modules (PAM)
- Chapter 34. Network Addresses
- Chapter 35. Network Devices and Configuration
- Chapter 36. Firewalls
- Chapter 37. System Startup and Shutdown
- Chapter 38. GRUB
- Chapter 39. System Init: systemd, SystemV and Upstart
- Chapter 40. Backup and Recovery Methods
- Chapter 41. Linux Security Modules
- Chapter 42. Local System Security
- Chapter 43. Basic Troubleshooting
- Chapter 44. System Rescue

I had been using Windows Subsystem for Linux (WSL) for a couple of years, so I could do some basic experimenting in that environment.

However, in order to be able to perform all the labs in the course I also needed some regular virtual machines, so I spun up 3 different distros in Azure in order to be able to try out the different package managers and such:

```powershell
Get-AzVM
ResourceGroupName        Name   Location          VmSize  OsType
-----------------        ----   --------          ------  ------
COMPUTE-RG         RHEL-AZ-01 norwayeast Standard_D2s_v3   Linux
COMPUTE-RG         SUSE-AZ-01 norwayeast Standard_D2s_v3   Linux
COMPUTE-RG         UBNT-AZ-01 norwayeast Standard_D2s_v3   Linux
```

Using the Auto-Shutdown feature for Azure VMs, this did not cost very much. In addition, I spun up an Ubuntu Server 20.04 in a local Hyper-V VM to make it easier to play around with GRUB configurations and system rescue techniques.

One very cool thing I noticed when deploying the latter is that PowerShell is listed as a Featured Server Snap:

![alt](/images/2021-02-25-ps-snap.png){:height="333px" width="600px"}

Although this is very convenient, as far as I know there is one gotcha of using PowerShell as a Snap package: SSH-based PowerShell remoting is not supported, as that relies on pwsh being available as a native binary - not inside a Snap package.

Since I have close to 20 years of experience managing Windows, I of course knew the basics of many of the topics outlined in the course. This helped a lot not having to spend too much time on the concepts, but rather on how to perform the task in this (for me) new environment.

The exam experience itself is performance based, where you get access to a terminal windows in a browser. This is very similar to the Katakoda playground, which is a free interactive playground with [many different environments](https://www.katacoda.com/learn). For example an [Ubuntu 20.04](https://www.katacoda.com/courses/ubuntu/playground2004) sandbox which becomes available in seconds.

On the left hand-side, you will have an overview of 24 tasks consisting of one or more sub-tasks. Given that you have 120 minutes available, there are 5 minutes available per task. This means that time management is key to success when taking the exam. I failed at this in my first attempt, as I spent too much time looking up information in man pages and had to rush through the last 5-6 questions without proper time to answer all of them. This was partly due to me not having the muscle memory regarding the various commands and topics needed, and partly due to the fact that the LFS 201 course in fact did not cover all of the topics measured. In the "What it prepares you for" section in the [course page](https://training.linuxfoundation.org/training/essentials-of-linux-system-administration/), the following is stated: "The topics covered are directly aligned with the knowledge domains tested by the Linux Foundation Certified Systems Administrator (LFCS) exam". This is something I discovered a few days ahead of the exam, by looking at various articles and threads online discussing the exam. Specifically [this thread](https://unix.stackexchange.com/questions/357940/lfcs-exam-vs-lfs201-training-topics/357956) was very surprising to read. Although the time I spent in the course was not wasteful by any means, there were additional topics needing attention and upskilling. Especially in the area Essential Commands which 25% of the exam is about.

I scheduled a second attempt 3 days later and started diving into the missing gaps. In addition I spent some time becoming more efficient in navigating the man pages. In the first attemp I spent too much time skimming through man pages by hitting Space. I did know about the search capability, where one can simply type / followed by a keyword and press Enter. However, what I did not know was the various navigation capabilities within man - such as using N to go to the next match. This is also a little detail that saves a lot of time - not only in the exam but also in real life. Another tip for efficiency is using the terminal multiplexer tmux - which is available and allowed to use during the exam. This makes it possible to create additional terminal windows, splitting them either vertically or horizontally. I typically used one window for looking up man pages or verifying contents in the output files requested in the various task, which made it easier to keep context in the main window.

After the first attempt, I felt rather confident I did not pass. After the second attempt, I was rather confident I did pass - and 36 hours after the exam I got an e-mail with the confirmation and certificate:

![alt](/images/2021-02-25-lfcs.jpg){:height="561px" width="600px"}


Another useful tool I can also recommend is [tldr](https://tldr.sh/) - which is simplified and community-driven man pages. This is not allowed during the exam as far as I know, as it is not included in the operating system. However, in day-to-day work it can be a time saver for quickly finding examples. This was initally one of the things I was missing coming from PowerShell, where Get-Help <some-new-command> -Examples is often what I look at to quickly grasp how a command can be used. Tldr is very much filling this gap for me.

It can be installed from the package manager for your distribution, for example on Ubuntu:
sudo apt install tldr

Example usage:

![alt](/images/2021-02-25-tldr.png)

Another tool, which is available in the operating system and hence allowed during the exam, is the info utility.

This can be leveraged to read info documents, and the syntax is *info command/concept*

For examle, here is the output of *info coreutils*

![alt](/images/2021-02-25-info.png)

As we can see we can navigate around in the document to read about the various topics. This is typically not something you will have time for during the exam, but the utility is very handy in day-to-day scenarios.

If you are looking to dive more into using PowerShell on Linux - and maybe also try to take the LFCS exam - feel free to reach out if you have any questions (comment field below, [@JanEgilRing on Twitter](https://www.twitter.com/janegilring) or jan.egil.ring (at) powershell.no) ).

## Useful resources

- [Installing PowerShell on Linux](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux)
- [PowerShell remoting over SSH](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/ssh-remoting-in-powershell-core)
- [PowerShell equivalents for common Linux/bash commands](https://mathieubuisson.github.io/powershell-linux-bash/)
- [Linux Foundation Certified System Administrator (LFCS)](https://training.linuxfoundation.org/certification/linux-foundation-certified-sysadmin-lfcs/)
- [Practice Test](http://mathematicbren.blogspot.com/2014/12/practice-test.html?m=1)