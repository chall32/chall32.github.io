---
layout: post
title: "Nutanix Kubernetes Platform Deployment - Part 1"
excerpt: "Container Management Without Getting Lost at Sea: Navigate the World of Kubernetes with Nutanix"
tags: 
- Nutanix
- Kubernetes
- Docker
- Deployment
image:
  thumb: nutanix-kubernetes-platform-deployment-pt1/nutanix-kubernetes-platform-deployment-pt1-01.png"
comments: true
date: 2025-06-09T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NKP" src="/images/nutanix-kubernetes-platform-deployment-pt1/nutanix-kubernetes-platform-deployment-pt1-01.png">
Kubernetes has evolved from a container orchestration curiosity into the backbone of modern cloud-native infrastructure. Yet for many organisations, the journey from "we need Kubernetes" to "we have production-ready Kubernetes" remains fraught with complexity, operational overhead, and the dreaded vendor lock-in. Enter the Nutanix Kubernetes Platform (NKP) - a solution that promises to deliver pure upstream, open-source Kubernetes with the operational simplicity that infrastructure teams actually want to work with.

NKP isn't just another managed Kubernetes offering wrapped in proprietary tooling. 

NKP provides a consistent experience for deploying and managing Kubernetes clusters at scale across on-premises, edge locations, and public cloud environments, all whilst maintaining that crucial upstream compatibility. The platform comes loaded with strategically selected best-of-breed infrastructure applications that are absolutely critical for running Kubernetes in production - think monitoring with Prometheus and Grafana, data protection with Velero, and comprehensive networking and storage solutions that just work out of the box. In this two-part series, we'll walk through building a solid foundation platform that's ready for NKP deployment, then in part two, we'll deploy and configure NKP itself. 

Because let's be honest - having a robust, well-planned infrastructure foundation is half the battle won when it comes to successful Kubernetes deployments.

{% include _toc.html %}
## Containers and Kubernetes
Before we dive into the world of Kubernetes platforms, let's take a step back and understand what we're actually dealing with. 

Think of containers as incredibly lightweight virtual machines - but instead of virtualising entire operating systems, they package up just your application and its dependencies into a portable, consistent unit. Imagine you're moving house: traditional virtual machines are like moving your entire house brick by brick, whilst containers are like packing everything into standardised shipping containers that can be moved anywhere and unpacked exactly as they were. This means your application runs the same way whether it's on your laptop, your test environment, or your production servers. The beauty of containers lies in their efficiency - you can run dozens of containerised applications on the same server that might only support a handful of traditional virtual machines.

Now, here's where Kubernetes comes into play. Managing a few containers is straightforward enough, but what happens when you have hundreds or thousands of containers that need to work together, scale up and down based on demand, recover from failures, and communicate securely?

That's where Kubernetes steps in as the conductor of this container orchestra. Kubernetes automates the deployment, scaling, and management of containerised applications across clusters of servers, handling everything from load balancing and service discovery to automated rollouts and rollbacks. It's evolved from a container orchestration curiosity into the backbone of modern cloud-native infrastructure.

## The Nutanix Kubernetes Platform (NKP)
A picture tells a thousand words:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NKP" src="/images/nutanix-kubernetes-platform-deployment-pt1/nutanix-kubernetes-platform-deployment-pt1-02.png">

As you can see - and as mentioned above - out of the box the P (Platform) in NKP is constructed using container based platform applications. For each of the service areas identified in the above diagram, the corresponding grey segment details the platform application(s) that will be used by NKP to provide the required service. 

Whilst many of these applications are familiar, to find out more about the individual applications see [Platform Applications](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_15:top-platform-apps-c.html){:target="_blank"} in the NKP documentation. Also see [Supported Platform Applications](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_15:top-nkp-supported-apps-c.html){:target="_blank"} for an applications per NKP license breakdown.

Yes, NKP can be deployed on multiple on-prem and public cloud platforms:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NKP cluster create help" src="/images/nutanix-kubernetes-platform-deployment-pt1/nutanix-kubernetes-platform-deployment-pt1-03.png">

What is Konvoy?<br>
*Konvoy, formerly known as D2iQ Kubernetes Platform (DKP), is a Kubernetes platform designed for enterprise use. It simplifies the deployment, management, and scaling of Kubernetes clusters across various environments, including on-premises data centers and public clouds. [D2iQ was purchased by Nutanix in January 2024](https://www.nutanix.com/blog/acquisition-of-d2iq-platform){:target="_blank"}.*

But perhaps we are getting ahead of ourselves...

## This Post: NKP Lab Deployment Preparation 
In this post I want to concentrate on preparing my lab infrastructure to run NKP. I'll be deploying NKP into a lab environment on... yes you guessed it... Nutanix. This environment has internet access, i.e. non Air-gapped. In part 2 I'll move onto deploying NKP. No promises and if there is sufficient demand, I may post about deploying NKP elsewhere. :wink: 

The NKP deployment process can be broken down into roughly ten steps:
<p></p>
<img style="display: block; margin-left: auto; margin-right: auto;" alt="NKP Installation Steps" src="/images/nutanix-kubernetes-platform-deployment-pt1/nutanix-kubernetes-platform-deployment-pt1-04.png">
<p align='right'>KIND = Kubernetes IN Docker</p>
In this post I'll cover steps one to three to get us ready to run nkp in step four next time.

I'll detail the basic requirements along with how I met them. You can also reference the [NKP on Nutanix Prerequisites Documentation](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_15:top-nutanix-basic-prerequisites-c.html){:target="_blank"} for the full details.

At the time of writing (June 2025), NKP v2.15.0 is the latest version available, released 15 May 2025.

## Requirements 

<div>
<style scoped>
table{
    margin: 0 auto;
    width: 95%;
    border-collapse: collapse;
    border-spacing: 0;
    border:1px solid #000000; }
th{
    text-align: center;
    border:1px solid #000000; 
    padding: 5px;}
td{
    text-align: center;
    border:1px solid #000000;
    padding: 5px;}
tr:nth-child(even) {
    background-color: #efefef;}
</style>
</div>

### Accounts

| Item             | Link     | Details / Notes |
| :---------------- | :------: | :------------- |
| Nutanix Portal Account | <ins>[Nutanix Portal](https://portal.nutanix.com){:target="_blank"}</ins> | Required for Nutanix Downloads |
| Docker Hub Account | <ins>[Docker Hub](https://hub.docker.com/){:target="_blank"}</ins> | Docker rate limit unauthenticated accounts, so a free account is recommended! |

<br>If you don't have access to the Nutanix Portal for downloads, checkout [Test Drive](/nutanix-kubernetes-platform-deployment-pt1/#nkp-test-drive) below.

### Infrastructure

| Item            | Link      | Details / Notes |
| :---------------- | :------: | :------------- |
| Prism Element Cluster AOS 6.5, 6.8 or later | <ins>[Nutanix Portal](https://portal.nutanix.com/page/downloads?product=nos){:target="_blank"}</ins> | I'll be deploying to an AOS 7.0.1 cluster, running AHV 10.0.1 as my hypervisor |
| Prism Central 2024.1 or later | <ins>[Nutanix Portal](https://portal.nutanix.com/page/downloads?product=prism){:target="_blank"}</ins> | I'll be deploying using a 2024.3.1.1 Prism Central |

<br>Remember, I'm going to deploy NKP on Nutanix. If deploying elsewhere, your requirements will be different. See [Infrastructure Provider-Specific Requirements](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_15:top-resource-req-provider-specific-c.html){:target="_blank"} for details.

### Software

| Item            | Link      | Details / Notes |
| :---------------- | :------: | :------------- |
| NKP for Linux | <ins>[Nutanix Portal](https://portal.nutanix.com/page/downloads?product=nkp){:target="_blank"}</ins> | Installation binary. I'll be using NKP 2.15.0 |
| NKP Node OS Image | <ins>[Nutanix Portal](https://portal.nutanix.com/page/downloads?product=nkp){:target="_blank"}</ins> | I'll be using Rocky Linux 9.5-release-1.32.3-20250430150550 for AHV  v2.15.0 |
| Debian 12 | <ins>[Debian.org](https://www.debian.org/CD/netinst/){:target="_blank"}</ins> | OS for NKP deployment machine. I'll be using Debian 12.11.0 |
| Docker | <ins>[Docker.com](https://docs.docker.com/engine/install/debian/){:target="_blank"}</ins> | For creating the NKP bootstrap cluster on the NKP deployment machine. I'll be using 28.2.2 |
| kubectl | <ins>[Kubernetes.io](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/){:target="_blank"}</ins> | For Kubernetes administration. I'll be using 1.33.1 |

<br>Versions listed above are for reference and are latest available at time of writing. No doubt you'll be using the same or newer versions.

### NKP Deployment Machine

| Item            | Detail     |
| :---------------- | :------ |
| CPU | 2x vCPUs, 2 Cores per vCPU |
| Memory | 6 GB | 
| Disk | 1x 150GB |
| OS | Debian 12 |

<br>[The NKP documentation](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Kubernetes-Platform-v2_15:top-nutanix-nkp-prerequisites-c.html){:target="_blank"} states: An x86_64-based Linux or macOS machine. Yes, you can use whatever 'flavour' of 64bit Linux you like.

I've been using Debian based distributions [for a while now](https://polarclouds.co.uk/ubuntu-910-ch-installation-guide/){:target="_blank"}, so I feel at home with Debian.

## NKP Deployment Machine Build Out
A standard Debian 12 amd64 [netinst](https://lecorbeausvault.wordpress.com/2021/09/21/beginners-guide-to-a-debian-netinstall-ncusrses-installer/){:target="_blank"} with SSH server and standard system utilities only. I'm building this VM on the Nutanix cluster that I'm planning to also run NKP on.

After OS installation and as Debian comes pretty bare-bones, lets get some prerequisite packages installed:
{% highlight shell %}
root@deploy:~# apt update
root@deploy:~# apt install zip unzip curl ca-certificates
{% endhighlight %}

Next, lets create an [SSH key pair](https://polarclouds.co.uk/lost-or-forgotten-passwords-nutanix/#creating-an-ssh-key-pair){:target="_blank"}. I'll leave the passphrase empty as this is a lab:
{% highlight shell %}
root@deploy:~# ssh-keygen -a 100 -t ed25519 -C "Chris @ PolarClouds"
{% endhighlight %}

Next, lets install Docker. As per Step five in the graphic above, Docker is used on the deployment machine to create the KIND (Kubernetes IN Docker) bootstrap cluster which is in turn used to create the NKP cluster.

Follows is a cut and paste from the Docker documentation [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/){:target="_blank"}:
{% highlight shell %}
# Add Docker's official GPG key:
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
{% endhighlight %}

Next, with Docker installed, let's test our docker account:
{% highlight shell %}
root@deploy:~# docker login -u <YOUR_DOCKER_USERNAME>
{% endhighlight %}

After supplying a password you should be greeted with a `Login Succeeded` confirmation. 

Next, lets install kubectl, the command-line tool used to interact with Kubernetes clusters. Follows is a cut and paste from [Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/){:target="_blank"}. We'll also move it to our bin folder: 
{% highlight shell %}
root@deploy:~# curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
root@deploy:~# chmod +x kubectl
root@deploy:~# mv kubectl /usr/local/bin
{% endhighlight %}

Finally, let's download the NKP binary from the [Nutanix Portal](https://portal.nutanix.com/page/downloads?product=nkp){:target="_blank"}:
{% highlight shell %}
root@deploy:~# curl -Lo nkp_linux_amd64.tar.gz "<PORTAL_DOWNLOAD_LINK>"
root@deploy:~# tar zxvf nkp_linux_amd64.tar.gz 
root@deploy:~# chmod +x nkp
root@deploy:~# mv nkp /usr/local/bin
{% endhighlight %}

## NKP Prerequisites Video 
My colleague [Winson Sou](https://www.linkedin.com/in/winsonsou/){:target="_blank"} has put a video together on installing the prerequisites required to deploy Kubernetes clusters with NKP:
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/6TO3v3kSBIE?si=T3eXIWoSI6wk76I4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<br>Winson is using Rocky 9.5 for his deployment machine as opposed to Debian, however the basics are just the same.

## NKP Test Drive
If you want to have a look around NKP, but don't have the time or inclination to build your own lab environment:

[<img style="display: block; margin-left: auto; margin-right: auto;" alt="NKP Test Drive" src="/images/nutanix-kubernetes-platform-deployment-pt1/nutanix-kubernetes-platform-deployment-pt1-05.png">](https://www.nutanix.com/en_gb/one-platform.launchtestdrive.json?type=nkp&lpurl=one-platform-nkp){:target="_blank"}

## Conclusion and Wrap Up
In this first part of our Nutanix Kubernetes Platform Deployment series, we've covered the essential prerequisites for installing NKP. From setting up your environment to creating a Nutanix Portal account, you're now well-equipped to embark on this exciting journey. Remember, container management doesn't have to be overwhelming – with Nutanix and Kubernetes, you'll be navigating the waters in no time.

That's it for part 1 of my NKP deployment series! In the next post, we'll dive deeper into the installation process itself, covering topics such as configuring your cluster and deploying workloads. Stay tuned for more insights on how to harness the power of Kubernetes with Nutanix.

This is part one of a multipart series. Other parts of the series can be found:

- [Nutanix Kubernetes Platform Deployment - Part 1](/nutanix-kubernetes-platform-deployment-pt1/): (This part) NKP Lab Deployment Preparation
- Nutanix Kubernetes Platform Deployment - Part 2: *Coming Soon!*

-Chris