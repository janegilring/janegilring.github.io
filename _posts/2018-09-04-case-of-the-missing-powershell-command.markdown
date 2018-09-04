---
layout: post
title:  "The case of the missing PowerShell command"
date:   2018-09-04 14:30:00 +0200
comments: true
categories: PowerShell,troubleshooting
---

## Challenge

An Azure Automation runbook was failing with the following error:
> The term 'Set-​AzureADUser' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again. (raised by: Get-Command)

The runbook was being invoked on a Hybrid Runbook Worker running Windows Server 2016 and PowerShell 5.1, which also had the latest AzureAD module installed. Even so, the runbook would fail with the above error message. Other runbooks which also use the same command was not experiencing any issues - hence this seemed like quite a mystery.

The issue was also reproducible by copying relevant parts of the rather large runbook to a small test-runbook.


## Solution

First step of troubleshooting was to log on to the Hybrid Runbook Worker and test whether the AzureAD module can be loaded and that Get-Command can find the missing command:

![alt](/images/2018-09-04_case_of_the_missing_command_01.png){:height="183px" width="600px"}

Very strange - this also fails, meaning we can rule out Azure Automation as a potential root cause.

Next step was to manually type the whole command, and see if it still fails.

Voilà!

![alt](/images/2018-09-04_case_of_the_missing_command_02.png){:height="117px" width="600px"}

In the past, I've been bitten by copying certificate thumbprints from the graphical Certificate viewer in Windows.
It' s not visible for the human eye, but there is a hidden character at the beginning of the thumbprint:

![alt](/images/2018-09-04_case_of_the_missing_command_03.png){:height="354px" width="450px"}

This will be carried on if the thumbprint is copied to for example a configuration file for an application - hence the application will fail to find the certificate.

A similar incident was happening here. As I moved the cursor one step to the right in the Set-AzureADUser command copied from the runbook, it would not move to the right after the dash until I pressed the right arrow key two times. I then deleted the hidden character using the delete key, and the issue was gone. After also removing the hidden character from the runbook, it succeeded as we would expect at this point.

I am not sure how the hidden character got in there in the first place, but I suspect that I copied the command from an example over at docs.microsoft.com and got the extra character without knowing it.

After mentioning the issue on the [PowerShell Virtual User Group on Slack](http://slack.poshcode.org/), Chris Dent added the hidden character to a [Pester test](https://floobits.com/PoshCode/PowerShell.Slack.com/file/chrisdent/TroublesomeCharacters.ps1:86) he is working on. This will make it possible to uncover hidden issues like this in a release pipeline, before the code ever gets to run in production.

## Summary

Hidden characters can be a real challenge when it comes to troubleshooting.
Typing out the full command manually during testing is a good practice to rule out hidden surprises like this.
Also, leveraging Pester tests to uncover "troublesome" characters might be a good practice.