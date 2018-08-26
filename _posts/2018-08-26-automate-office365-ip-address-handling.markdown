---
layout: post
title:  "Automate Office 365 endpoint ACL configurations using PowerShell"
date:   2018-08-26 21:30:00 +0200
comments: true
categories: Exchange Online,Office 365,PowerShell
---

## Challenge

When configuring firewalls and access lists (ACLs) for allowing traffic to and from Office 365 services, keeping them up to date can be a challenge.

The Microsoft-article [Office 365 URLs and IP address ranges](https://docs.microsoft.com/en-us/office365/enterprise/urls-and-ip-address-ranges) contains an overview over which endpoints to include in the access lists - for example for Exchange Online:

![alt](/images/2018-08-26_Office365_ACLs_01.png){:height="310px" width="600px"}

 Some organizations handles both initial configuration, as well as keeping the ACLs up-to-date, manually. This is error prone, since the list of endpoints is quite long, and it takes a lot of time keeping track of changes and updating the configuration.

 I was running into this struggle myself after moving my Exchange server lab to Azure. I use Exchange Online Protection for mail-filtering, hence I needed to allow TCP port 25 from all of the Exchange Online Protection IP-ranges.

## Solution

There are many options available for publishing internal resources in Azure Virtual Networks to the internet, such as [Azure Firewall](https://docs.microsoft.com/en-us/azure/firewall/overview).
Since I had a basic lab scenario and wanted to keep things simple, I configured an [Azure Load Balancer](https://docs.microsoft.com/en-us/azure/load-balancer/load-balancer-overview) with an inbound network address translation (NAT) rule to forward TCP port 25 to the internal Exchange server. In order to restrict which IP addresses is allowed to connect to the internal server, I had to leverage Windows Firewall, since an Azure load balancer is a Layer-4 (TCP, UDP) load balancer.

Windows Firewall can be managed using PowerShell, so the next step was the challenge stated above - getting the IP addresses for Exchange Online Protection dynamically and configure the Windows Firwall accordingly.

In the past, I've seen PowerShell functions leveraging an online XML-document provided by Microsoft.
However, in the beforementioned article [Office 365 URLs and IP address ranges](https://docs.microsoft.com/en-us/office365/enterprise/urls-and-ip-address-ranges), there is a note about a new Web service available:

> Microsoft is developing a REST-based web service for the IP address and FQDN entries on this page. This new service will help you configure and update network perimeter devices such as firewalls and proxy servers. You can download the list of endpoints, the current version of the list, or specific changes. This service will eventually replace the XML document linked from this page. To try out this new service, go to Web service.

On the [Web service](https://docs.microsoft.com/en-us/office365/enterprise/managing-office-365-endpoints#webservice) page, the following information is provided:

> To help you better identify and differentiate Office 365 network traffic, a new web service publishes Office 365 endpoints, making it easier for you to evaluate, configure, and stay up to date with changes. This new web service (now in preview), will eventually replace the lists of endpoints in the Office 365 URLs and IP address ranges article, along with the RSS and XML versions of that data. These format of endpoint data are planned to be phased out on October 2, 2018.

A REST-based web service is a perfect match for PowerShell. Before starting to create a function from scratch, I found an excellent script created by [Joerg Hochwald](https://twitter.com/jhochwald), which he as written about [here](https://hochwald.net/powershell-get-the-office-365-endpoint-information-from-microsoft/).

I converted it to a function (personal preference) and put it in my toolbox (available [here](https://github.com/janegilring/PSCommunity/blob/master/Office%20365/Get-Office365Endpoint.ps1)).

The following Gist shows how to restrict Microsoft Exchange to only accept SMTP connections from Exchange Online Protection based on data retrieved by the Get-Office365Endpoint function:

{% gist e43b1146910458c774f85f253e2bc7e6 %}

The IP addresses is available as an array, ready to be fed as input into Set-NetFirewallAddressFilter:

![alt](/images/2018-08-26_Office365_ACLs_02.png){:height="442px" width="405px"}

After running the above example, the Windows Firewall rules is restricted to the Exchange Online Protection IP addresses:

![alt](/images/2018-08-26_Office365_ACLs_03.png){:height="575px" width="428px"}

Keeping the firewall rules up-to-date is now as simple as scheduling the code shown in the Gist as a scheduled task (or ideally, from Azure Automation).


## Summary

In this article we have looked at how to automate the configuration of Office 365 endpoint ACL configurations using PowerShell.  
Windows Firewall might be a trivial example, but enterprise products from vendors such as F5 Networks (BIG-IP), Sophos, and others provides REST API's which can be leveraged from PowerShell as we have seen in this article.