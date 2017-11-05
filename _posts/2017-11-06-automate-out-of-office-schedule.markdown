---
layout: post
title:  "Automate Out of Office scheduling using PowerShell"
date:   2017-11-05 16:00:00 +0200
comments: true
categories: Outlook,Exchange,PowerShell
---

## Challenge

If you have an Exchange mailbox, you have the option to configure Out of Office messages when you are away from work:  
![alt](/images/2017-11-06_automate_oof_01.png){:height="567px" width="600px"}

Usually, the default options is sufficient: You configure a message, a time range, and you are all set.  
In my scenario, I am having a paternity leave for the next 20 weeks where I will work 50% during the work week. This means I will have to turn Out of Office on and off several times every week.

## Solution

This is a perfect opportunity to look into what options we have as end users when working with PowerShell against Exchange - whether it is Exchange Online or an on-premises such as Exchange Server 2016.

The article [Connect to Exchange Online PowerShell](https://technet.microsoft.com/en-us/library/jj984289(v=exchg.160).aspx) outlines the necessary steps. Let us try this using a regular user with no administrative permissions and see which commands we have access to:

{% gist 93b3f751f851f239e746cec4088b2cdf %}

Sample output:  
![alt](/images/2017-11-06_automate_oof_02.png){:height="291px" width="600px"}

That is quite a lot of commands, and many of them can be very useful. For example, we can change our photo using Get/Set-UserPhoto as well as view and clean up mobile devices using Get/Set-MobileDevice.

What we are looking for is [Set-MailboxAutoReplyConfiguration](https://technet.microsoft.com/en-us/library/dd638217(v=exchg.160).aspx), which can perform the task we want to automate:

{% gist 379d23507fc0b40960ebc8e9455bc875 %}

Sample output:  
![alt](/images/2017-11-06_automate_oof_03.png){:height="113px" width="600px"}

We can also verify the result:  
![alt](/images/2017-11-06_automate_oof_04.png){:height="567px" width="600px"}

The last piece of the puzzle is to schedule this script to run on the desired days of week.  
One option is to run it as a scheduled task on a local computer, but I have chosen to run it in Azure Automation in order to avoid relying on a computer being powered on. You can get an Azure Automation account with up to 500 minutes of runbook run time each month, which should cover the needs in this example.

Sample runbook:

{% gist 7c731974fe910a032b7a5d784ecdd366 %}

The runbook can then the attached to a custom schedule in order to make it run on the week days we want:

![alt](/images/2017-11-06_automate_oof_05.png){:height="735px" width="300px"}

![alt](/images/2017-11-06_automate_oof_06.png){:height="730px" width="300px"}


## Summary

In this article we have looked at how to automate the configuration of Out of Office scheduling for an Exchange mailbox using PowerShell and Azure Automation.  
I am also using an Azure Automation runbook with the opposite schedule as for the Out of Office scheduling runbook for starting the climate control in my car. More details about that is available in a previous article - [Manage Tesla climate control system using PowerShell](http://www.powershell.no/homeautomation/2017/02/19/manage-tesla-climate-control-system-using-powershell.html).