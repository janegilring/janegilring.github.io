---
layout: post
title:  "Introducing the Azure File Sync DSC resource"
date:   2019-08-11 17:30:00 +0200
comments: true
categories: PowerShell, Azure
---

In this article, we are going to introduce a new PowerShell DSC resource module called AzureFileSyncDsc. Before we get started, let us have a quick overview of what Azure File Sync is.

# Azure File Sync

Azure File Sync is a cloud service which makes it possible for customers using Windows File servers to centralize file services in Azure storage, while having a cache in multiple locations for fast, local performance. Among the benefits of using the service is offloading local datacenter resources, such as backup infrastructure - by leveraging Azure Backup. It can also be a replacement for DFS-R (Distributed File System Replication).

The service is based on several building blocks:

- One or more Windows File servers (Windows Server 2012 R2 or newer)
- An agent deployed to the Windows file servers (referred to as Server Endpoints)
- An Azure storage account
- An Azure File Share (referred to as the Cloud Endpoint)
- An Azure Storage Sync service
- Optionally, an Azure Recovery Services Vault (for backing up Azure File shares)

A short introduction video is available here:
{% include youtubePlayer.html id="Zm2w8-TRn-o" %}

More information about the concept, requirements and planning considerations is available in the [documentation](https://docs.microsoft.com/en-us/azure/storage/files/storage-sync-files-planning).

I would also recommend the following video from Microsoft Ignite 2018 which provides a walkthrough of the features in Azure File Sync:

## Azure File Sync DSC resource
