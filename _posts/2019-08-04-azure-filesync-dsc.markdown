---
layout: post
title:  "Introducing the Azure File Sync DSC resource module"
date:   2019-08-04 17:30:00 +0200
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
{% include youtubePlayer.html id="HAQajyFJ58U" %}

## Azure File Sync DSC

The Azure File Sync DSC resource module contains a couple of DSC resources, which can provide declarative automation of some of the tasks required to install and configure the Azure File Sync agent on Windows Server.

The first one is installing the Azure File Sync agent and the Az PowerShell module, which is a prerequisite for the agent. The resource will also register the agent against the specified Azure Storage Sync service.

Example usage of the AzureFileSyncAgent DSC resource:

{% gist 87b812bbb1dbfc1035a9d843a1b86f41 %}

The second resource is adding a server endpoint, which will synchronize a cloud endpoint against a local path specified in the configuration.

Example usage of the AzureFileSyncServerEndpoint DSC resource:

{% gist 8eae732b07d0f99306ba7b36d8fc0ff1 %}

By combining other existing DSC resources, such as the StorageDsc module, it is possible to fully automate the setup of a hybrid file server from formatting disks, configuring drive letters to installing and registering the Azure File Sync agent - as well as adding a server endpoint. This is not only useful for initial setup, but also for disaster recovery purposes. Imagine a local file server fails for some reason and needs to be rebuilt. By combining fast VM provisioning with PowerShell Desired State Configuration for automating all aspects of the file server (adding file shares and such), the server can be provisioned in minutes and be ready to server client requests. Content will be downloaded on-demand when users tries to access files.

At PowerShell Conference Europe 2019 in Hannover, I did a presentation on Azure File Sync - where I showed the DSC resource in action. The recording is available here:
{% include youtubePlayer.html id="Qc9fXuavYXA" %}

If you encounter any issues, or have suggestions for improvements or new features, feel free to submit an issue in the [AzureFileSyncDsc repository](https://github.com/janegilring/AzureFileSyncDsc/issues) on GitHub.
