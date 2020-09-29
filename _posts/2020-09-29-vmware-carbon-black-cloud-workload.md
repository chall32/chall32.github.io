---
layout: post
title: "Introducing VMware Carbon Black Cloud Workload" 
excerpt: "Comprehensive Protection for vSphere Workloads"
tags: 
- VMware
- Pro-Tip
image:
  thumb: vmware-carbon-black-cloud-workload/vmware-carbon-black-cloud-workload00.png
comments: true
date: 2020-09-29T00:00:00+00:00
---
Just announced at VMworld 2020!
<a href="https://www.vmworld.com/en/index.html" target="_blank"><img style="display: block; margin-left: auto; margin-right: auto;" alt="VMworld 2020" src="/images/vmware-carbon-black-cloud-workload/vmworld2020.png"></a>

<img style="float: right; margin: 0px 0px 10px 10px;" alt="carbon black cloud workload" src="/images/vmware-carbon-black-cloud-workload/vmware-carbon-black-cloud-workload01.png">
If you think about your typical deployment today, you've probably got the following infrastructure deployed:

1. An estate of workload VMs running various tasks and (possibly) operating systems
2. Cluster(s) of virtualisation hosts to host your workload VMs
3. Management/orchestration server(s) to manage your virtualisation hosts
4. An update manager to manage the patching of your virtualisation hosts

Chances are you are achieving the above today via VMware vSphere; ESXi for 2. and vCenter for 3. and 4. above. 

**Question:** how do you handle the security of your workload VMs?

Let's look at the 2020 Gartner market guide for cloud workload protection platforms:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Workload Security Requirements" src="/images/vmware-carbon-black-cloud-workload/vmware-carbon-black-cloud-workload02.png">

Chances are today you are running several disparate solutions outside of VMware vSphere solution to handle some or all of the the above requirements. What if you could combine these disparate tools into a single solution? 

This is where vMware Carbon Black Cloud Workload comes in.

{% include _toc.html %}
## What is VMware Carbon Black Cloud Workload?
VMware Carbon Black Cloud Workload address modern day security challenges in the following ways:

- Integrates with existing vSphere infrastructure
- Easy deployment and lifecycle management
- Replace legacy antivirus to reduce agents on production workloads
- Increase visibility of workloads
- Provide asset context via audit and/or remediation
- Provide focus on common exploits and high-risk vulnerabilities
- Help IT operationalize security
- Provide shared visibility for infrastructure and Security teams
- Make it easy to respond quickly to incidents and vulnerabilities

Breaking this down further VMware Carbon Black Cloud Workload can:
### Identify Risk
- Risk-prioritized vulnerability assessment of workloads in vCenter
- Workload inventory status visibility in vCenter
- Query over 2000 workload artefacts on-demand
- Run ongoing assessments to track infrastructure hygiene
- Take immediate action with live, remote access  
 
### Prevent
- Stop advanced malware
- Shut down file-less attacks
- Easily adapt prevention
- Replace Legacy antivirus on workload VMs

### Detect and Respond
- Enhance visibility for Security and IT teams
- Detect anomalous activity
- Feed response actions into hardening and prevention
- Correlate attacks to vulnerabilities via alerts

## In Action
As this has only just been announced, screenshots of Carbon Black Cloud Workload in action are a little scarce. However, I've managed to snag the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="In action 1" src="/images/vmware-carbon-black-cloud-workload/vmware-carbon-black-cloud-workload03.png">

From the above you can see the Carbon Black Cloud Workload plugin installed into vCenter and the VMware admin is taking a look at their 136 infrastructure assets - VMs, ESXi hosts and vCenters. From there they can see that across those 136 assets they have 27 critical vulnerabilities that they need to address.

Drilling into one of the detected vulnerabilities:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="In action 2" src="/images/vmware-carbon-black-cloud-workload/vmware-carbon-black-cloud-workload04.png">

From the above we can see that our VMware admin can understand vulnerability context along with with risk score. There are also links to the [National Vulnerability Database](https://nvd.nist.gov/).

What's more is that the VMware admin managed to gather all of the above information without having to leave the comfort of their vSphere client.

## Versions
Carbon Black Cloud Workload is available in the following versions:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Versions" src="/images/vmware-carbon-black-cloud-workload/vmware-carbon-black-cloud-workload05.png">
Abbreviations:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Abbreviations" src="/images/vmware-carbon-black-cloud-workload/vmware-carbon-black-cloud-workload06.png">

## Free Trial? - Why Yes!
What's more, I would like to bring your attention to this:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Special Offerings" src="/images/vmware-carbon-black-cloud-workload/vmware-carbon-black-cloud-workload07.png">

Yep, there will be  a free trial of Carbon Black Cloud Workload Essentials available for current vSphere and vCloud Foundation customers through to April 2021!

## Conclusion and Wrap Up
Given VMware's purchase of Carbon Black back in October 2019, the writing was on the wall for VMware to move into the security arena. Carbon Black Cloud Workload is a culmination of that purchase. 

As I have already mentioned throughout this post, Carbon Black Cloud Workload is a brand new product announced by VMware at VMworld 2020.  Therefore details are a little scarce at present.

### Busy
So lets see. Over and above the 15 reasons why your VMware admin once again became your most valuable asset I gave seven years ago back in 2013 [HERE](https://polarclouds.co.uk/vmware-component-integration/) (which doesn't include network admin via NSX and storage admin via vSAN), your VMware admin is now even more valuable to your business; your VMware admin is now very much an integral member of your security team too!

Busy times for us VMware admins!! :sunglasses:

Back to Carbon Black Cloud Workload - I'll try to post updates to this post when more information becomes available. :thumbsup:

In the meantime, checkout the [VMware Carbon Black Cloud Workload product information site](https://www.carbonblack.com/products/vmware-carbon-black-cloud-workload/).

-Chris