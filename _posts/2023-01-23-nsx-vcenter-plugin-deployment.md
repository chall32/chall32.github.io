---
layout: post
title: "NSX vCenter Plug-in Deployment" 
excerpt: "vCenter and NSX Management Back Together Again"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-00.png
comments: true
date: 2023-01-23T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-00.png">
In this post we will look at installing NSX as a plug-in to vCenter server, as promised [a while ago](/nsx-t-3-2-manual-microsegmentation/){:target="_blank"}.

Historically, NSX-T or [just NSX](https://blogs.vmware.com/partnernews/2022/04/nsx-data-center-name-change.html){:target="_blank"} as it is now known as was installed as a separate entity and managed away from vCenter and the vSphere client. 

Since the releases of vSphere 7.0 Update 3 and NSX 3.2.0, NSX can now be installed and managed in the vSphere client via a vCenter plug-in in much the same way as the previous VMware network virtualisation product, [NSX-v used to be](/nsx-data-center-failover-pt2/#disconnect-secondary-nsx-manger-from-primary){:target="_blank"}.

First off, let's take a look around our environment prior to NSX deployment.

{% include _toc.html %}
## vSphere Environment
vSphere 8.0 environment consisting of one ESXi 8.0 and one vCenter 8.0 server:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Environment 1" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-01.png">

A single version 8.0 vSphere Distributed Switch (VDS) handling all connectivity via two uplink NICs:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Environment 1a" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-01a.png">

VDS topology:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Environment 2" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-02.png">

## NSX Deployment
OK, let's download the correct NSX installer from the [VMware software portal](https://customerconnect.vmware.com/en/downloads/info/slug/networking_security/vmware_nsx/4_x){:target="_blank"}. In this article I'll be deploying NSX v4.0.1.1 into my vSphere 8.0 environment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select NSX Manager vCenter Plugin" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-03.png">

Next, from the vSphere Client menu, select the **NSX** option:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select NSX" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-04.png">

The install wizard starts. Select **Install NSX**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select Install NSX" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-05.png">

Supply location of downloaded NSX Manager with vCenter Plug-in OVA file:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Find Downloaded File" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-06.png">

Name the VM and select an inventory location for the VM to reside:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Name VM" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-07.png">

Select the ESXi Host (if [DRS](https://www.vmware.com/uk/products/vsphere/drs-dpm.html){:target="_blank"} not enabled for your cluster):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select Compute Resource" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-08.png">

Confirm access to vCenter Server:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Confirm vCenter Access" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-09.png">

Select NSX Manager VM deployment size. For sizing requirements, see [NSX 4.0 Manager VM Resource Requirements](https://docs.vmware.com/en/VMware-NSX/4.0/installation/GUID-AECA2EE0-90FC-48C4-8EDB-66517ACFE415.html#nsx-manager-vm-resource-requirements-3){:target="_blank"}:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select Manager Size" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-10.png">

Select datastore for the NSX Manager VM:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select Storage" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-11.png">

Select management network for the NSX Manager VM:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select Network" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-12.png">

Complete the required information. At a minimum, you need to provide NSX Manager:
 - Root account password
 - Hostname
 - IP address details (if not using DHCP)
 - DNS details 
 - NTP details

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Complete Details" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-13.png">

Select vCenter to associate NSX with:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Associate vCenter" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-14.png">

Review and finish NSX manager deployment wizard:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Ready to Complete" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-15.png">

Allow some time for the NSX-T manager VM to complete it's first boot activities and initial configuration. In normal circumstances this can take around 15 to 20 minutes to complete.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Start NSX Onboarding" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-16.png">

After refreshing the browser to load the NSX plug-in, Select **NSX** from the vSphere Client menu again:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select Installed NSX Plug In" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-17.png">

Welcome indeed! Let's supply our licence key:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Licence Key" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-18.png">

Finally we arrive at the Getting Started Wizard page:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select one of the recommended use cases to configure NSX" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-19.png">

As a check, yes we can also browse and login directly to our newly deployed NSX manager:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX Manager Interface" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-20.png">

## Options
For those who ***don't wanna follow no crazy wizard!!1!*** - that is those of a more hardcore or "back to basics" type of disposition, wishing to learn about the NSX installation process and the configuration that makes up an NSX deployment from the ground up by doing it manually, check out the following:

**Manual NSX Installation: Security Only** (also known as a Micro-Segmentation installation):
 - [NSX-T 3.2: Micro-Segmentation Only Deployment - Manual Setup](/nsx-t-3-2-manual-microsegmentation/){:target="_blank"} 

**Manual NSX Installation: Virtual Networking** (also known as an Overlay installation):
 - [NSX-T 3.2: Overlay Lab Build - Part 1](/nsx-t-overlay-lab-pt1/){:target="_blank"}
 - [NSX-T 3.2: Overlay Lab Build - Part 2](/nsx-t-overlay-lab-pt2/){:target="_blank"}
 
**Automated NSX Installation:** For those that don't want to follow an NSX wizard **OR** complete a manual install:
- [NSX-T 3.2: Overlay Lab Build - Part 3](/nsx-t-overlay-lab-pt3/){:target="_blank"} 

Where we complete an NSX virtual networking installation using PowerShell and NSX API calls alone!

Options, options. Plenty of options!

## Conclusion and Wrap Up
That will do it for this post. In upcoming posts we will follow the security only and virtual networking wizards to complete our NSX installs.

-Chris