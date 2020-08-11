---
layout: post
title:  "Getting into the world of containers"
date:   2020-08-11 19:10:00 +0200
comments: true
categories: DevOps, Kubernetes, Docker, containers
---

# Getting into the world of containers

Up until the past year or so I have mainly been working with Windows-based systems. However, as technology evolves there is a need to adapt - and getting into the Linux side of things is one key area given the use of Linux in a variety of technologies.

Bryan Liles had a great [tweet](https://twitter.com/bryanl/status/1287409603307810819) on this topic recently:

![alt](/images/2020-08-11_CKA_01.png){:height="347px" width="600px"}

When working in IT, continous learning is a fundamental requirement.

Like the introduction of virtual machines was a revolution in the IT industry 20 years ago, the introduction of containers is a very similar transition. After some initial adoption time, it became the norm. The way I see it, the same is now happening with containers.

Traditionally when working with software, a common challenge has been getting different software with different dependencies working on the same operating system - being Windows Server, a Linux distribution or other systems - especially when different versions of dependencies has been required.
Containers solves this challenge by packaging and isolating the entire runtime environment.
Container technology is not something new, as it has existed on the Linux/Unix side for decades. There are multiple container runtimes available, such as [containerd](https://containerd.io/), although the most popular one is [Docker](https://www.docker.com/).
Another important concept in the container world is *container registries* - a central location to push and pull container *images*. This provides a great portability, as container images can be pulled and run basically anywhere - cloud environments such as Amazon Web Services, Microsoft Azure or on-premises.

Containers is also an important aspect of microservices, and commonly used in lously coupled and independently deployable microservice architectures:

![alt](/images/2020-08-11_CKA_04.png)
*Image credit: [Red Hat](https://www.redhat.com/en/topics/microservices/what-are-microservices)*

Although microservices is not the answer to each and every scenario, if your monolith works - it might not be any point in changing for the sake of changing. However, if you are either starting from scratch or looking into modernise and migrate an on-prem monolith to a more scalable cloud environment, a microservice architecture using containers as one of it's building blocks is highly relevant. One of the Kubernetes co-founders, Brendan Burns, had a great talk on this topic on .NET Conf 2020 called *[Why You Should Care About Microservices
](https://www.youtube.com/watch?v=7hY6fggwHqU&feature=youtu.be)* which digs deeper into this topic with some great advice.

Even though containers is great, there is also a need for *orchestration*. A mechanism which can start/stop, distribute and provide the needed resiliceny and scaling of applications running in containers. This is where [Kubernetes](http://kubernetes.io/) comes into the picture - a container orchestration system for deploying, scaling and managing containerized applications.

Kubernetes has become a de-facto standard for container orhestration in a similar way that Docker has become a de-facto container runtime.

A quote from Microsoft`s whitepaper ["Kubernetes Learning Path"](https://azure.microsoft.com/mediahandler/files/resourcefiles/kubernetes-learning-path/Kubernetes%20Learning%20Path_Version%202.0.pdf):

*"Kubernetes is taking the app development world by storm. By 2022, more than 75% of global organizations will be running containerized applications in production."*

This statement is backing up that getting into the world of containers is an important step when working with IT infrastructure in the 2020s.

## Certified Kubernetes Administrator certification

In the past year I've been exposed to both [Red Hat OpenShift](https://www.openshift.com/) (a container application platform based on Kubernetes) and [Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/aks/) working in customer projects, so I felt it was time to sharpen my skills and aim for the [Certified Kubernetes Administrator(CKA)](https://www.cncf.io/certification/cka/) certification to gain a base level of skills on Kubernetes.

The exam was a very different experience than what I'm used to from various Microsoft certifications, since all of it was performance based - no questions.

When taking the exam, you get access to a web-based terminal window where you have 3 hours to solve 24 practical tasks/problems.

Everything is running on Ubuntu Server, so for me I also had to invest in learning some core Linux skills.

Every task has a weight such as 2% or 7%, and you need at least 74% in order to pass the exam.

There is a lot of learning resources online. What I chose was to read the book [Kubernetes Up and Running](https://azure.microsoft.com/en-us/resources/kubernetes-up-and-running/) in addition to going through 6 excellent [PluralSight courses](https://www.pluralsight.com/paths/certified-kubernetes-administrator) from Anthony Nocentino. Earlier this year I've also attended cloudskill.io's excellent [Docker](https://portal.cloudskills.io/docker-jumpstart) and [Kubernetes](https://portal.cloudskills.io/k8s) courses with the great instructors Dan Wahlin, Mike Pfeiffer and again Anthony Nocentino.

A core requirement before taking the exam is to get hands-on experience and play around with the technology - this can't be stressed enough. Personally I like to get some theory in place first in order to understand the core concepts, and then start playing around.

There are a lot of options on that side as well:
- **[Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)** *is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop for users looking to try out Kubernetes or develop with it day-to-day.*
- **[Kind](https://kubernetes.io/docs/setup/learning-environment/kind/)** *is a tool for running local Kubernetes clusters using Docker container "nodes".*
- **[Kubernetes in Docker Desktop](https://www.docker.com/products/kubernetes)**  *- the Docker client application for MacOS and Windows machines for the building and sharing of containerized applications and microservices.*
- **[Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal)** - managed control plane (master nodes), making it very easy to get a cluster up and running.
- **[Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/)** - also a managed control plane offering
- **[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine)** - a managed control plane offering from the company who invented Kubernetes. (Fun fact: Kubernetes is the successor of the Google internal large-scale infrastructure systems Borg and Omega).
- **Manual deployment on either virtual or physical servers** - a good starting point is Kelsey Hightower's guide [*Kubernetes The Hard Way*](https://github.com/kelseyhightower/kubernetes-the-hard-way) and Rudi Martinsen's article *[Setting Up a Kubernetes Cluster on virtual machines](https://rudimartinsen.com/2020/08/08/setting-up-a-kubernetes-cluster-vms/)*.


In addition, there is a number of (free) online services available for playing with Kubernetes using interactive browser-based scenarios. I've used [Katakoda](https://www.katacoda.com/courses/kubernetes) many times, both for going through their labs but also when wanting quick access to check or test something ad-hoc.


![alt](/images/2020-08-11_CKA_02.png)
*Screenshot of the Katakoda web terminal*

Regarding the exam and certification itself there are a number of online articles with tips and tricks. My main take-away is time management: Even though you have 3 hours available, they will fly fast if you get stuck on a task and spend a lot of time trying to solve it. I would recommend going through the tasks as fast as possible and solve the low-hanging fruits first - then come back to the harder ones. Also make sure to prioritize those with the highest weight (5%+).

Before taking the real exam, I would recommend trying a simulator such as killer.sh - where you in addition to get a score can get answers with detailed explanations.

Note that you do get 1 free retake when ordering the exam. I was a bit too ambitious when racing through the mentioned book and online courses in 3 weeks, so I got a score of 71% on the first attempt and realized I needed to practice a bit more on a couple of subjects. On the retake August 10th, I passed with a score of 87%.

![alt](/images/2020-08-11_CKA_03.jpg)

Good luck if you also aim for the CKA exam. If not, I would highly recommend to get to know the basics of containers if you work in the IT infrastructure space regardless of the certification.

## Useful resources

- [Kubernetes documentation](https://kubernetes.io/docs/)
- ["Kubernetes Basics" YouTube-series](https://www.youtube.com/playlist?list=PLLasX02E8BPCrIhFrc_ZiINhbRkYMKdPT) by Kubernetes co-founder Brendan Burns
- [Kubernetes Slack community](https://slack.k8s.io/) - Tip for fellow Norwegians: There is a *#norw-users* channel)
- [The Azure Kubernetes Service Checklist](https://www.the-aks-checklist.com/) -  ensure your cluster is production-ready
- [Awesome-Kubernetes](https://github.com/ramitsurana/awesome-kubernetes) - A curated list for awesome kubernetes sources
- [Official K8s Certification Candidate Handbook](https://training.linuxfoundation.org/wp-content/uploads/2019/08/CKA-CKAD-Candidate-Handbook-8.5.19.pdf )
- [CKA Exercises
](https://github.com/stretchcloud/cka-lab-practice) - A set of exercises that will help you to prepare for the Certified Kubernetes Administrator exam, offered by the Cloud Native Computing Foundation
