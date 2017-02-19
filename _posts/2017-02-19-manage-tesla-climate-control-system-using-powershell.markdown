---
layout: post
title:  "Manage Tesla climate control system using PowerShell"
date:   2017-02-19 17:33:50 +0200
comments: true
categories: HomeAutomation
---

# Challenge

Tesla's vehicles has a climate control system which can be remotely managed from the company`s mobile app.  
Many Tesla owners use this to pre-heat or pre-cool the cabin remotely before using the car.  

![alt](/images/Tesla-logo.png){:height="150px" width="186px"}

A frequently requested but missing feature is the ability to schedule the state of the climate control system.


# Solution

Since this article probably will be read by a less technical audience than what I  expect for other 
articles on this blog, I will start by giving some definitions for relevant parts of the proposed solution.

**PowerShell**
 
>PowerShell (including Windows PowerShell and PowerShell Core) is a task automation and configuration management framework from Microsoft, consisting of a command-line shell and associated scripting language built on the .NET Framework.

*Source:* [Wikipedia](https://en.wikipedia.org/wiki/PowerShell)

PowerShell was previously available on Windows only, but last year Microsoft  [made PowerShell open source](https://blogs.msdn.microsoft.com/powershell/2016/08/18/powershell-on-linux-and-open-source-2/) and available on both Linux and MacOS.
The Linux and MacOS versions is still in preview, but the Windows version has been built-in to Windows since Windows 7.

**Scheduled task**  

>Task Scheduler is a component of Microsoft Windows that provides the ability to schedule the launch of programs or scripts at pre-defined times or after specified time intervals.

*Source:* [Wikipedia](https://en.wikipedia.org/wiki/Windows_Task_Scheduler)

This means that a PowerShell script can be scheduled to run at pre-defined intervals, which is what 
 we want to accomplish in this article.

**Azure Automation**  

>Microsoft Azure Automation provides a way for users to automate the manual, long-running, error-prone, and frequently repeated tasks that are commonly performed in a cloud and enterprise environment. It saves time and increases the reliability of regular administrative tasks and even schedules them to be automatically performed at regular intervals. You can automate processes using runbooks. A runbook is a set of tasks that perform some automated process in Azure Automation.

*Source:* [Microsoft](https://docs.microsoft.com/en-us/azure/automation/automation-intro)

Azure Automation has a free plan which gives you 500 minutes of runbook execution time per month,  
which should be more enough for the task we are trying to solve in this context.

The main benefit of using Azure Automation in this context is that we can schedule a PowerShell  
script to accomplish a task without being dependent on a computer being powered on at the time the scheduled task is configured to run.

## Controlling your Tesla vehicle from PowerShell

PowerShell can be extended by installing modules which adds commands for a specific  
task or product. You can think of the [PowerShell Gallery](https://www.powershellgallery.com) as an "app-store" for 
 PowerShell, and makes it very convenient to add new modules by simply running a command.

**1. Start PowerShell (on Windows 7 - 10)**  

Simply press the Start button and search for "PowerShell". You will likely get two hits:
"Windows PowerShell" and "Windows PowerShell ISE". The first one is a command console while  
the PowerShell ISE also has a script editor. If you're a beginner it might be easier  
to start with the PowerShell ISE, as it provides helpful features such as IntelliSense.  
It also has a script editor, which is useful when building a script which we will do  
in this article.

**2. Allow PowerShell scripts to be executed**  

PowerShell has a feature called "execution policy" which by default is set to "Restricted",  
meaning that no scripts is allowed to run. In the context of this article, I will recommend  
to set the execution policy to "RemoteSigned". This means that you can run scripts locally  
without having to sign it with a digital signature.

Run the following command to configure the execution policy:  
*Set-ExecutionPolicy RemoteSigned*

Make sure you start PowerShell with "Run As Administrator" before running the command.

**3. Install the Tesla module for PowerShell**  

A PowerShell module created by Jon Newman makes it possible to run PowerShell commands  
which uses the same API as when using the Tesla mobile app and controlling the vehicle  
from mytesla.com. If you are interested in the technical details, you can find more  
information in the module's [GitHub repository](https://github.com/JonnMsft/TeslaPSModule).

The module is available from the PowerShell Gallery, meaning we can install it by simply running the following:  
*Install-Module -Name Tesla* 

If this is the first time you run this command, you will be prompted to install NuGet which is being 
 used under the hood to interact with the PowerShell Gallery. Answer Yes when prompted to install 
this prerequisite. Next, you will be warned that the PowerShell Gallery by default is configured  
as an untrusted source. Answer Yes to acknowledge this and install the module.

Now the module is installed and is ready to be used.  
Here is an example on how to explore the available commands and authenticate using your MyTesla  
credentials:

![alt](/images/2017-02-19_Tesla_PowerShell_01.png){:height="500px" width="244px"}
  
Next, we can run the API command get_climate to get the current climate settings:

![alt](/images/2017-02-19_Tesla_PowerShell_02.png){:height="320px" width="328px"}
  
Here we can see that we have two API calls for starting and stopping the auto conditioning:
![alt](/images/2017-02-19_Tesla_PowerShell_03.png){:height="500px" width="245px"}

Based on what we have seen so far, we have enough information to create a simple script to start and 
stop auto conditioning:

{% gist 1af6d498f02abcfa43129d75fee3ff3a %}

Since we are using PowerShell, only the imagination can limit what criterias we can use before  
executing a command such as starting or stopping heating. For example, we have already seen how we  
can get information about the current climate state fromt he vehicle including the outside temperature.  
This means we can use logic in PowerShell's scripting language for starting heating only if the  
outside temperature is below a given value:

{% gist ffc7c531683320a63ccc26ffcde425f9 %}

I have created a [repository on GitHub](https://github.com/janegilring/Tesla) for storing examples on how you can use  
PowerShell to manage your Tesla vehicle.  
This will be updated over time with more examples, and hopefully others will contribute  
as well. If you have any suggestions for new features, feel free to submit an [issue](https://github.com/janegilring/Tesla/issues)  
on the GitHub repository.

## Scheduling the PowerShell script

**Option 1 - scheduled task in Windows**

**Option 2 - runbook in Azure Automation**

**Option 3 - others**  

Scheduled tasks and scheduled Azure Automation runbooks is two examples, but there are a number of  
other options available for scheduling PowerShell scripts.
Feel free to leave your suggestions in the comment field below. Maybe you did something completely  
different? I'd  love to hear about it.
