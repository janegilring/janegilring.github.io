---
layout: post
title:  "Getting started with AWS from an Azure perspective"
date:   2021-08-01 15:00:00 +0200
comments: true
categories: AWS, Azure, Cloud, DevOps
---

# Introduction

In recent years I have been working with the Microsoft Azure cloud platform at many customers, and gained experience among several of the many services available.
As I have been expanding my horizon and learned new technologies such as containers (Docker and Kubernetes), Infrastructure as Code (Terraform) and operating systems (Linux) in the past couple of years - I also wanted to learn more about multiple cloud providers.
Not necessarily because I want to change my focus from Azure, but in order to gain more insight into other providers - and maybe get some hands-on and real world experiences as well.

![alt](/images/2021-08_azure_aws.png)

As I have a couple of customers who are likely going into multi cloud scenarios with both Azure and AWS (for various reasons), I found it to be a good opportunity to spend time learn the basics of AWS and aim for a certification.
During my studies towards the **[AWS Certified Solutions Architect – Associate](https://aws.amazon.com/certification/certified-solutions-architect-associate/)**, I made several interesting findings with regards to similarities and differences between Azure and AWS and wanted to share some of my perspective on this topic based on my study notes.

In the Azure Application Architecture Guide, Microsoft has an [Azure for AWS professionals](https://docs.microsoft.com/en-us/azure/architecture/aws-professional/) section, which contains comparisons between various services in Azure and AWS. The tables outlined in the sections below is sourced from that documentation, with my own observations at the end.


## Compute

| AWS service                           | Azure service              | Description                                                                  |
| :------------------------------------ | :------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Elastic Compute Cloud (EC2) Instances | Virtual Machines           | Virtual servers allow users to deploy, manage, and maintain OS and server software. Instance types provide combinations of CPU/RAM. Users pay for what they use with the flexibility to change sizes.                       |
| Batch                                 | Batch                      | Run large-scale parallel and high-performance computing applications efficiently in the cloud.                                                         |
| Auto Scaling                          | Virtual Machine Scale Sets | Allows you to automatically change the number of VM instances. You set defined metric and thresholds that determine if the platform adds or removes instances.  |
| VMware Cloud on AWS                   | Azure VMware Solution      | Seamlessly move VMware-based workloads from your datacenter to Azure and integrate your VMware environment with Azure. Keep managing your existing environments with the same VMware tools you already know while you modernize your applications with Azure native services. Azure VMware Solution is a Microsoft service, verified by VMware, that runs on Azure infrastructure. |
| Parallel Cluster                      | CycleCloud                 | Create, manage, operate, and optimize HPC and big compute clusters of any scale |

***My observations***

- *Auto Scaling in AWS manages standard EC2 instances, while VM Scale Sets in Azure manages scale set child virtual machines which uses a separate API than regular VMs. A new VM Scale Set feature called [Flexible orchestration mode](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes#what-has-changed-with-flexible-orchestration-mode) is currently in Public preview, and provides orchestration features over standard Azure IaaS VMs, instead of scale set child virtual machines. Also, when you create a VM and add it to a Flexible scale set, you have full control over instance names within the Azure Naming convention rules.*
- *Spread Placement Groups in AWS = Availability Sets in Azure*
- *AWS has it's own Linux distro: [Amazon Linux 2](https://aws.amazon.com/amazon-linux-2/), which is an LTS Kernel tuned for better performance on Amazon EC2. Microsoft also have it's own Linux distro, [CBL-Mariner](https://github.com/microsoft/CBL-Mariner). However, it is not a general purpose distribution, but highly specialized for use internally in the Azure infrastructure. [This article](https://blog.jreypo.io/2021/07/09/a-look-into-cbl-mariner-microsoft-internal-linux-distribution/?s=09) contains more in-depth information. A quote:*

>CBL-Mariner follows the secure-by-default principle, most aspects of the OS have been built with an emphasis on security. It comes with a hardened kernel, signed updates, ASLR, compiler-based hardening and tamper-resistant logs amongst many features.
CBL-Mariner consumes limited disk and memory resources. The lightweight characteristics of CBL-Mariner also provides faster boot times and a minimal attack surface.
However, let’s make an important clarification: CBM-Mariner is not a general purpose Linux distro. Its purpose is to be used as an internal lightweight Linux distro for Microsoft’s engineering teams into the Azure infrastructure.

## Containers and container orchestrators

| AWS service                      | Azure service            | Description                                                                                                                                                                                   |
| :------------------------------- | :----------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Elastic Container Service (ECS), Fargate  | Container Instances      | Azure Container Instances is the fastest and simplest way to run a container in Azure, without having to provision any virtual machines or adopt a higher-level orchestration service.        |
| Elastic Container Registry       | Container Registry       | Allows customers to store Docker formatted images. Used to create all types of container deployments on Azure.                                                                                |
| Elastic Kubernetes Service (EKS) | Kubernetes Service (AKS) | Deploy orchestrated containerized applications with Kubernetes. Simplify monitoring and cluster management through auto upgrades and a built-in operations console. See AKS solution journey. |
| App Mesh                         | Service Fabric Mesh      | Fully managed service that enables developers to deploy microservices applications without managing virtual machines, storage, or networking.                                                 |

***My observations***

- *Having worked quite a bit with Kubernetes on Azure in the past year, it was interesting to see the mapping of the same services in AWS. Although I haven't tested the AWS services in-depth yet, the only main difference I noticed during my studies was that AWS also offers an option to run containers on EC2 instance by leveraging an agent inside the VMs. The similar service in Azure, Container Instances, have a single option - which is to run the Container Instances on managed infrastructure - which means no VMs. Fargate provides a similar experience on the AWS side - where there are no EC2 instances to manage.*
- *Container techology really provides major benefits when it comes to multi cloud architectures. A great example is showcased in [the third episode of the “Cloud stories from Norway”](https://pulse.microsoft.com/nb-no/work-productivity-nb-no/na/fa2-recap-of-cloud-stories-from-norway-episode-3-onboarding-to-kubernetes-and-building-resilient-services-on-azure-in-nrk-content/) videoshow. [NRK](https://en.wikipedia.org/wiki/NRK) - a Norwegian radio and television public broadcasting company and the largest media organization in Norway - shows how they run infrastructure across 2 cloud providers as well as their own datacenter by leveraging containers and Kubernetes.*

## Databases

| Type                | AWS Service                           | Azure Service                                           | Description                                                                                                                                             |
| :------------------ | :------------------------------------ | :------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Relational database | RDS                                   | SQL Database, Database for MySQL, Database for PostgreSQL | Managed relational database service where resiliency, scale, and maintenance are primarily handled by the platform.                                     |
| NoSQL / Document    | DynamoDB, SimpleDB, Amazon DocumentDB | Cosmos DB                                               | A globally distributed, multi-model database that natively supports multiple data models: key-value, documents, graphs, and columnar.                   |
| Caching             | ElastiCache                           | Cache for Redis                                         | An in-memory–based, distributed caching service that provides a high-performance store typically used to offload nontransactional work from a database. |
| Database migration  | Database Migration Service            | Database Migration Service                              | Migration of database schema and data from one database format to a specific database technology in the cloud.                                          |

***My observations***

- *Amazon Aurora is a MySQL and PostgreSQL-compatible relational database part of the Amazon Relational Database Service. It was interesting to learn that this offering was initially developed and highly cloud optimized for internal use, and later offered to customers.*

## Networking

| Area                        | AWS service                 | Azure service       | Description                                                                                                                                                                                                                                           |
| :-------------------------- | :-------------------------- | :------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cloud virtual networking    | Virtual Private Cloud (VPC) | Virtual Network     | Provides an isolated, private environment in the cloud. Users have control over their virtual networking environment, including selection of their own IP address range, creation of subnets, and configuration of route tables and network gateways. |
| Cross-premises connectivity | VPN Gateway                 | VPN Gateway         | Connects Azure virtual networks to other Azure virtual networks, or customer on-premises networks (Site To Site). Allows end users to connect to Azure services through VPN tunneling (Point To Site).                                                |
| DNS management              | Route 53                    | DNS                 | Manage your DNS records using the same credentials and billing and support contract as your other Azure services                                                                                                                                      |
|                             | Route 53                    | Traffic Manager     | A service that hosts domain names, plus routes users to Internet applications, connects user requests to datacenters, manages traffic to apps, and improves app availability with automatic failover.                                                 |
| Dedicated network           | Direct Connect              | ExpressRoute        | Establishes a dedicated, private network connection from a location to the cloud provider (not over the Internet).                                                                                                                                    |
| Load balancing              | Network Load Balancer       | Load Balancer       | Azure Load Balancer load balances traffic at layer 4 (TCP or UDP). Standard Load Balancer also supports Availability Zones.                                                                                                        |
|                             | Application Load Balancer   | Application Gateway | Application Gateway is a layer 7 load balancer. It supports SSL termination, cookie-based session affinity, and round robin for load-balancing traffic.                                                                                               |

***My observations***

- *When creating a new Virtual Private Cloud in AWS - similar to a Virtual Network in Azure - unlike in Azure, there are no internet connectivity by default. An internet gateway will need to be created and attached to the desired subnets. An exception is the [Default VPC and default subnets](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html), which do have an Internet gateway configured and according to the documentation is "suitable for getting started quickly, and for launching public instances such as a blog or simple website".*
- *In AWS, a subnet is tied to Availability Zone (note: subnet, not VPC). This is a key difference from Azure, where a Virtual Network including subnets spans across Availability Zones.*
-  *In AWS, Security Group rules are based on ALLOWs and there is no concept of DENY when in comes to Security Groups. This means you cannot explicitly deny or blacklist specific ports via Security Groups, you can only implicitly deny them by excluding them in your ALLOWs list*
- *In AWS, Security groups control inbound and outbound traffic for your instances (they act as a Firewall for EC2 Instances) while NACLs control inbound and outbound traffic for your subnets (they act as a Firewall for Subnets). In Azure, a Security Group can be bound to either a virtual machine network interface or a subnet. In addition, Azure have the concept of Deny in Security Groups.*
- *Features in AWS Route53 is supported in Azure, but spread across multiple services such as Azure DNS, Azure Private DNS and Azure Traffic Manager.*


## Object storage

| AWS service                  | Azure service | Description                                                                                                                                             |
| :--------------------------- | :------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Simple Storage Services (S3) | Blob storage  | Object storage service, for use cases including cloud applications, content distribution, backup, archiving, disaster recovery, and big data analytics. |

## Virtual machine disks

| AWS service               | Azure service | Description                                                                                                               |
| :------------------------ | :------------ | :------------------------------------------------------------------------------------------------------------------------ |
| Elastic Block Store (EBS) | managed disks | SSD storage optimized for I/O intensive read/write operations. For use as high-performance Azure virtual machine storage. |

***My observations***

- *By default, an EC2 instance with an attached AWS Elastic Block Store (EBS) root volume will be deleted together when the instance is terminated. This differs from Azure, where both the OS-disk and data disks is kept when deleting a virtual machine.*
-   *You can change EBS volumes on the fly, including the size and storage type. In Azure, a virtual machine must be shutdown in order to resize both the OS disk as well as data disks.*
- EBS Multi Attach in AWS = Shared Disk in Azure, and is only supported  for io1/io2 tiers.

## Shared files

| AWS service         | Azure service | Description                                                                                                                                                                |
| :------------------ | :------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Elastic File System | Files         | Provides a simple interface to create and configure file systems quickly, and share common files. Can be used with traditional protocols that access files over a network. |

***My observations***

- *AWS Transfer Family has AWS Transfer for FTP, FTPS and SFTP supporting LDAP, AD, Cognito and custom authentication mechanisms. Currently, there are no SFTP (and FTPS) protocol support for Azure storage, but according to [this](https://feedback.azure.com/forums/217298-storage/suggestions/33001027-sftp-and-ftps-protocol-support-for-azure-files?page=1&per_page=20) UserVoice item, Microsoft are working on this and will share their plans for this offering soon.*
- *Lifecycle management is supported for EFS in AWS, but Files in Azure does not currently support this.*
- *AWS EFS supports Linux (NFS) only. For Windows, there is a separate service called Amazon FSx for Windows File Server - which is managed EC2 instances under the hood. Azure Files on the other hand, is a new implementation of SMB on the highly scalable Azure storage platform. Also, AWS’ offer is SSD-based only, whereas Azure Files is available in both premium (SSD) and standard (HDD) - which might make it more cost effective in certain scenarios.*


## Archiving and backup

| AWS service               | Azure service               | Description                                                                                                   |
| :------------------------ | :-------------------------- | :------------------------------------------------------------------------------------------------------------ |
| S3 Infrequent Access (IA) | Storage cool tier           | Cool storage is a lower-cost tier for storing data that is infrequently accessed and long-lived.              |
| S3 Glacier, Deep Archive  | Storage archive access tier | Archive storage has the lowest storage cost and higher data retrieval costs compared to hot and cool storage. |
| Backup                    | Backup                      | Back up and recover files and folders from the cloud, and provide offsite protection against data loss.       |

## Hybrid storage

| AWS service     | Azure service | Description                                                                                                                                                            |
| :-------------- | :------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Storage Gateway | StorSimple    | Integrates on-premises IT environments with cloud storage. Automates data management and storage, plus supports disaster recovery.                                     |
| DataSync        | File Sync     | Azure Files can be deployed in two main ways: by directly mounting the serverless Azure file shares or by caching Azure file shares on-premises using Azure File Sync. |

## Bulk data transfer

| AWS service                                       | Azure service | Description                                                                                                                                    |
| :------------------------------------------------ | :------------ | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| Import/Export Disk                                | Import/Export | A data transport solution that uses secure disks and appliances to transfer large amounts of data. Also offers data protection during transit. |
| Import/Export Snowball, Snowball Edge, Snowmobile | Data Box      | Petabyte- to exabyte-scale data transport solution that uses secure data storage devices to transfer large amounts of data to and from Azure.  |


## Miscellanous

***Observations***

- **Naming** of resources (such as virtual machines) is a bit different in AWS, as they do get automatic IDs (for example, i-10a64379) by default and there is no requirement to specify a name. However, one can optionally add tags such as Name - and it will appear in the Name column in the AWS portal.
- **Resource groups** are quite different in the two providers. Quote from the [Resource management on Azure and AWS](https://docs.microsoft.com/en-us/azure/architecture/aws-professional/resources) section in the Azure for AWS professionals:

>Both Azure and AWS have entities called "resource groups" that organize resources such as VMs, storage, and virtual networking devices. However, Azure resource groups are not directly comparable to AWS resource groups.While AWS allows a resource to be tagged into multiple resource groups, an Azure resource is always associated with one resource group. A resource created in one resource group can be moved to another group, but can only be in one resource group at a time. Resource groups are the fundamental grouping used by Azure Resource Manager.

- **Management interfaces**: Both cloud providers provides multiple interfaces including a web interface, REST APIs, command line utilities and templating capabilities, as one would expect. Personally, I was particularly interested in the AWS PowerShell module, which is structured into multiple sub-modules per service as with the Azure PowerShell module. To give an idea how the experience with the AWS PowerShell module looks like, here is a snippet from my initial explorations:

```powershell
Set-AWSCredential  -AccessKey "<get-from-lastpass>" -SecretKey "<get-from-lastpass>" -StoreAs janegilring-iam-user
Set-AWSCredential -ProfileName janegilring-iam-user
Set-DefaultAWSRegion -Region eu-north-1 # Stockholm

Install-Module -Name AWS.Tools.Common
Get-Command -mod AWS.Tools.Common
Get-AWSCredential -ListProfileDetail

Install-Module -Name AWS.Tools.Installer -Force
Install-AWSToolsModule AWS.Tools.EC2,AWS.Tools.S3

Get-AWSPowerShellVersion
Get-AWSPowerShellVersion -ListServiceVersionInfo

Update-AWSToolsModule -CleanUp

Get-AWSService

$amiId = Get-SSMLatestEC2Image -Path ami-windows-latest -ImageName Windows_Server-2019-English-Core-EKS_Optimized-1.17
$instance = New-EC2Instance -ImageId $amiId -InstanceType t3.medium -SubnetId $subnetId -KeyName jerlabkeypair -AssociatePublicIp $false -SecurityGroupId $securityGroup
```

The [AWS Tools for PowerShell](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-welcome.html) was greatly enhanced after several former PowerShell team members joined AWS, where things like performance and memory usage was enhanced by modularizing the module into sub-modules per service as well as enabling auto-importing of the AWS.Tools cmdlets.

- **Infrastructure as Code**: Like Azure have it's Azure Resource Manager templates (ARM templates), AWS have Cloudformation which describes your desired resources and their dependencies so you can launch and configure them together as a so-called stack. An overview from the [documentation](https://aws.amazon.com/cloudformation/):

![alt](/images/2021-08-CloudFormation.png)

Both ARM and Cloudformation leverages JSON (or alternatively YAML for Cloudformation), which can be cumbersome to work with.

To mitigate this and make IaC easier to working, Azure created [Bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) - a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources.

An alternative which many companies use for IaC is Terraform, which in addition to having providers for both AWS and Azure have many other providers - for example for Kubernetes. Contininuing on that example, one could provision a Kubernetes cluster in either cloud provider and use the Terraform Kubernetes provider to do configuration of Kubernetes specific elements inside the cluster after provisioning the infrastructure. This might make sense in many scenarios, especially in multi-cloud environments where the same underlying templating language can be used.

On the DevOps side of things, similarly to Azure DevOps, which contains many services such as Azure Repos for source control and Azure Pipelines for Continuous Integration/Continous Deployment, AWS has the corresponding AWS Code Commit and AWS CodePipeline services.

- **Managed services**: Both providers have several managed service offerings, where customers can run services as PaaS. One such example is AWS Managed Active Directory and Azure Active Directory Domain Services, which provides similar capabilities.

- A successful cloud application will focus on five pillars of software quality: **Cost optimization, Operational excellence, Performance efficiency, Reliability, and Security**.
Microsoft have the [Azure Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/
) to assess your architecture across these five pillars. Similarly, AWS have the [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/wellarchitected-framework.pdf). Both of these is well worth reading, as they provide useful guidance and insights into the topics outlined.

Lastly, I would like to recommend Mike Pfeiffer's video [
AWS for Azure Pros: The Ultimate AWS to Azure Service Comparison](
https://www.youtube.com/watch?v=m94tlNJCjSI), which provides great insight into the foundational differences between the two providers. Having worked in AWS, as well as working a lot with Azure these days, Mike have a lot of useful insight to share.


## Exam tips

How we learn the best varies from person to person. For me, a combination of video lectures and hands-on lab exploration works great.

I started out following along on the project-focused [AWS Cloud Architect Bootcamp](https://cloudskills.io/courses/aws-cloud-architect) on CloudSkills.io earlier this year.

When ramping up my studies, I went through Stephane Maarek's online course [AWS Certified Solutions Architect Associate 2021](https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c02) (on a 1.5x speed). This was a great asset, where I performed lab explorations while following along. It was also convenient that the course is available via the Udemy app, so I could watch through some lectures when I found a spare moment.

Another useful resource was the [AWS SAA-C02 Study Guide](https://github.com/keenanromain/AWS-SAA-C02-Study-Guide) by Keenan Romain. This guide contains a lot of compressed information about the various exam objectives.

The last tip is to go through a number of practice exams. I did not manage to find time for that before taking the exam after a couple of intense weeks of studies, but I have seen many people recommend the exams from [Tutorials Dojo](https://portal.tutorialsdojo.com/courses/aws-certified-solutions-architect-associate-practice-exams/).

## Summary

I [passed](https://www.credly.com/badges/834d9c2d-d60e-4f0d-a8da-f978ea131efe/linked_in) the exam, and it has been very interesting to learn more about the similarities and differences between Azure and AWS during my exam studies. Even if working with one primary cloud provider, I think it is useful to get to know at least the basics and what capabilities other providers have. Also, given that more and more companies is moving towards multi cloud infrastructures, you might be facing such scenario sooner rather than later. Reasons for this can be many, such as company mergings or strategic decisions to spread the risk across providers for enhanced business continuity.

![alt](/images/2021-08-AWS-SolArchitect-Associate.png)