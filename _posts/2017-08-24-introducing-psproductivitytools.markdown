---
layout: post
title:  "Announcing the PowerShell Productivity Tools module"
date:   2017-08-24 22:47:20 +0200
comments: true
categories: Productivity
---

In this article, we are going to introduce a new PowerShell module called PSProductivityTools.

![alt](/images/2017-08-24_Productivity_01.jpg){:height="387px" width="600px"}


**Introducing the PowerShell Productivity Tools module**

The module is published to the PowerShell Gallery, which means you can install it using the following command from the PowerShellGet module:

*Install-Module -Name PSProductivityTools*

If you do not have the option to leverage the PowerShellGet module, manual installation instructions as well as information about requirements is available in the [GitHub repository](https://github.com/janegilring/PSProductivityTools) for the module.

The initial version of the module contains the following functions:
- **Start-Pomodoro** - Initiates a new Pomodoro sprint and supports several actions such as configuring availability in Skype for Business, enable presentation mode, start music and trigger custom tasks using IFTT such as muting/unmuting a mobile device.
- **Publish-SfBContactInformation** - Publish-SfBContactInformation is a PowerShell function to configure a set of availability settings in the Skype for Business client.


**Time management**

![alt](/images/2017-08-24_Productivity_02.png)

Time management is something most people is having to face every day. At work, we try to be as productive as possible. However, it may not be easy due to many distractions during a work day.  
Many books are written on this topic, and several techniques exists for trying to put your available time into a system. The [Pomodoro technique](https://cirillocompany.de/pages/pomodoro-technique) is one of the more popular time management techniques. It is a great way to eliminate distractions, such as phone notifications, e-mail, questions from collegues and so on. In a nutshell, the technique uses a timer to put bulks of work into several sprints - typically lasting for 25 minutes.

A while back, [MVP Ståle Hansen](http://twitter.com/StaleHansen) wrote a PowerShell script for starting a Pomodoro sprint using PowerShell as the timer. In addition to starting a timer, he also invoked additional actions such as enabling presentation mode to avoid notifications to pop-up on the computer.  
A few years ago I wrote a PowerShell function for publishing Skype for Business (formerly Lync) availability using a PowerShell function. Ståle incorporated this into his script as well. After talking together about how useful the Start-Pomodoro function is, we decided to create a PowerShell module hosted on GitHub and published to the PowerShell Gallery in order to make it easier for the community to install, use and contribute.

Here is some examples on how to leverage the Start-Pomodoro function:
{% gist 04c07f3c0ddaf95a475be2b77338fce5 %}

For additional information, I would recommend to read Ståle`s article [Set yourself unavailable with this open source PowerShell based Pomodoro timer](https://msunified.net/2017/08/23/set-yourself-unavailable-with-this-open-source-powershell-based-pomodoro-timer/).

Any other suggestions are also welcome, feel free to submit an [issue](https://github.com/janegilring/PSProductivityTools/issues) on the GitHub repository.

**Productivity devices**

In addition to the Start-Pomodoro PowerShell function, a few physical tools can be of great help as well.

My first recommendation is to get a good headset with noice cancellation capabilities.
I recently bought the [Sennheiser MB 660 UC MS Wireless Headset](www.sennheiser.com/wireless-headset-office-phone-mb-660-uc-ms), which I highly recommend:

![alt](/images/2017-08-24_Productivity_04.png){:height="600px" width="600px"}

Another device I recently bought and is quite happy with after a couple weeks of usage is [Blynclight](https://embrava.com/products/blync-light?variant=328886579) - a "busy light" which indicates with colors what your availability state is based on your Skype for Business status:

![alt](/images/2017-08-24_Productivity_03.png)


**Summary**

In this article we have looked at the PowerShell Productivity Tools module - which contains commands for productivity tools & topics such as time management. Going forward we would like to both expand the functionality of the existing commands as well as add new productivity related ones. If you have an idea, we would love to hear it even if you do not plan to implement it yourself.  
Hopefully, the PowerShell Productivity Tools module can help you be more productive during busy work days.