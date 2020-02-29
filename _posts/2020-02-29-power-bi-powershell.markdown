---
layout: post
title:  "Getting data from Power BI using PowerShell"
date:   2020-02-29 08:30:00 +0200
comments: true
categories: Power BI, PowerShell
---

In this article, we are going to have a look at how to access data in Power BI from PowerShell.

## Scenario

The scenario I was working with before writing this article was a business process needing a notification based on a threshold for a value available in a tile in a Power BI report.

The data was being synchronized from an on-premises database into a Power BI data set on an hourly schedule.

Based on this data set, a dashboard and a report was created in Power BI to gain insight into business processes. The requirement in this case was to alert via multiple channels (e-mail, SMS and Microsoft Teams) based on certain conditions.

I was asked to assist getting the value out of a dashboard tile using PowerShell in order to configure an automated process using Azure Automation and Logic Apps as building blocks.

Having helped out with setting up an Azure Automation runbook for scheduled refresh of data sets in Power BI, I had some insight into the official [Microsoft Power BI PowerShell module](https://docs.microsoft.com/en-us/powershell/power-bi/overview?view=powerbi-ps). Hence I started looking around there for a way to retrieve information from the Power BI dashboard tile in question:

{% gist a96a577d1ebc8602044e060f7b8f548d %}

However, none of the Power BI cmdlets seemed to accomplish what we wanted.
There is a [REST API for Power BI](https://docs.microsoft.com/en-us/rest/api/power-bi/), along with a generic Invoke-PowerBIRestMethod cmdlet taking care of authentication.

We started to look into the following resource in the API:
https://api.powerbi.com/v1.0/myorg/reports/{reportId}/pages/{pageName}

However, this turned out to only return metadata about the pages in the report. Not the actual data values from tiles.

It turns out the Power BI PowerShell module is only applicable to the management plane in Power BI - hence it cannot be used to retrieve the actual data. This is similar to working with a resource in Azure like Azure SQL for example: The Az PowerShell module makes it possible to manage all aspects of the Azure SQL Management plane (create logical servers and databases). However, to access the actual content in the database, we must work with the data plane and rather use the [SqlServer PowerShell module](https://docs.microsoft.com/en-us/sql/powershell/sql-server-powershell?view=sql-server-ver15) if the goal is to retrieve some SQL-data using PowerShell.

## The solution

For accessing the data plane in Power BI, there is an XMLA endpoint available.

The Power BI team has written a great article called [Power BI open-platform connectivity with XMLA endpoints](https://powerbi.microsoft.com/en-us/blog/power-bi-open-platform-connectivity-with-xmla-endpoints-public-preview/), where the following client tools for working with the XMLA endpoints is layed out:

| Tool                                       | Description                                                                                                                                                            |
|--------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SQL Server Data Tools                      | Model authoring with a range of enterprise features, integration with source control, and application-lifecycle management processes.                                  |
| SQL Server Management Studio               | Perform fine-grain data refresh, scripting and management.                                                                                                             |
| Tabular Editor                             | Open-source, community tool with extensive set of enterprise modelling features, and easy-to-use experience                                                            |
| Power BI  ALM Toolkit and  BISM Normalizer | Application-lifecycle management, incremental deployments and model merging as described in this whitepaper                                                            |
| Others                                     | Analysis Services has a rich history of open-source community tools. Various third-party software vendors provide tools for monitoring and managing Analysis Services. |

PowerShell would fall into the "Others" category, where we have the option to use the SqlServer PowerShell module. In that module, there is an Invoke-ASCmd cmdlet, which according to the [documentation](https://docs.microsoft.com/en-us/powershell/module/sqlserver/invoke-ascmd?view=sqlserver-ps) *Enables a database administrator to execute an XMLA script, Multidimensional Expressions (MDX) query, or Data Mining Extensions (DMX) statement against an instance of Microsoft SQL Server Analysis Services.*.




{% gist ada89d75ceb8cc558cd9738873c4b36b %}

This suits our needs perfectly in this scenario, as we can use DMX queries similar to the ones used when building reports and dashboards to query the data.

In the end, we can see the complete picture regarding how the end result looks like:

![alt](/images/20200229_PowerBI_Scenario.png)





## Summary

In this article we have seen how to retrieve data from a data set in Power BI, as well as seen the importance of understanding whether to use the management or data plane to get the necessary information.
