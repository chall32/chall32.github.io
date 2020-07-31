---
layout: post
title: "VMware NSX Data Center for vSphere Failover/Failback - Part 1" 
excerpt: "Why and Getting Familiar"
tags: 
- VMware
- Pro-Tip
image:
  thumb: nsx-upgrade/nsx-upgrade-00.jpg
comments: true
date: 2020-07-31T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX" src="/images/nsx-upgrade/nsx-upgrade-00.jpg">

Whilst the newer version of VMware NSX, NSX-T is gaining some traction in the wider community of late, it has yet to reach the level of business adoption that NSX Data Center (formally/affectionately known as NSX-v) has. There are still many, many organisations running NSX Data Center.

With that in mind, I wanted to post a series of articles performing a site failover and recovery of my NSX Data Center test lab.
{% include _toc.html %}
## Why?
As can be seen below, an NSX Data Center installation requires several components to operate:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX Components" src="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-00.jpg">

Working down from the top of the image -
### Management Plane
These components consist of vCenter and NSX Manager. Typically these are "doubled up" - that is they are deployed on each site of an NSX for Data Center solution/environment.

### Control Plane
These components consist of our NSX Controller cluster and our Universal Logical Distributed Router (UDLR) control VMs. 

The control plane VMs can only exist on **ONE SITE**, typically the primary site. Because they can only exist on one site, **in the event of loss of the site containing the control VMs, they must be recreated at a secondary site to reinstate the NSX control plane.** 
{: style="color:red"}

### Data Plane 
These components consist of the ESXi hosts and the NSX Edge (also known as Edge Service Gateway or ESG) VMs. Again, typically these are "doubled up" - that is they are deployed on each site of an NSX for Data Center solution/environment.

## Types of Failover
Essentially there are two types of failover scenario:

### Planned Failover
The primary site is going offline for an extended period of time.<br>
For example a relocation of the site to a new building. Any service outages are known about ahead of time and can be planned for and mitigated as much as possible up front.

### Unplanned Failover
The primary site is suddenly offline and will remain so for an extended period of time.<br>
It is what it is. Pieces need picking up and any service outages need to be addressed as soon as possible.

## Getting Familiar with the Lab
We will be performing failover in the following lab:
<a target="_blank" href="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-01.png"><img style="display:block;" src="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-01.png" alt="NSX Test Lab"/></a><sup>(Click image to zoom in)</sup>
### Lab Items Worthy of Note
- Two site model with vCenters and NSX Managers deployed at each site
- Site A is the primary site
- Site B is the secondary site
- Site A houses NSX controller cluster and the UDLR control VMs
- "UNIVERSAL-SITE-A" and "UNIVERSAL_SITE_B" represent the universal layer 2 LANs spanning both A and B sites
- VMs plugged in to "UNIVERSAL-SITE-A" use ESG-SITE-A as their preferred north/south egress/ingress point
- VMs plugged in to "UNIVERSAL-SITE-B" use ESG-SITE-B as their preferred north/south egress/ingress point
- BGP peering is used between UDLRs, Edges and LABROUTER (a pfSense router) for dynamic routing

## Failover Prerequisites
The following should ideally be configured / captured prior to a failover event:
- Ensure that [Controller Disconnected Operation (CDO) mode](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.admin.doc/GUID-9302DCCA-12E9-409D-858E-110A91639A69.html) is enabled on all NSX Managers in the environment
- Setup an IP Pool for the NSX Controllers on the secondary site
- UDLR configuration captured - including interfaces, ECMP status, static routes (if any) and BGP config
- Admin credentials for all ESGs, UDLRs, NSX Managers and vCenters at both sites

## Component Placement After a Failover
<a target="_blank" href="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-02.png"><img style="display:block;" src="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-02.png" alt="Site A Failed"/></a><sup>(Click image to zoom in)</sup><br>
Following a failover of Site A to Site B, the following can be observed:
- NSX Controller cluster is rebuilt on Site B 
- Universal Site A UDLR control VM is rebuilt on Site B
- Universal Site B UDLR control VM is rebuilt on Site B

Additionally, Site B NSX Manager promoted to primary.

## Conclusion and Wrap up
That'll do it for part one.

In this part we looked at failure scenarios, became familiar with our NSX test lab. We also looked at prerequisites to enable a smooth failover. Over the next couple of posts we will get into performing a primary site failover along with a failback once our failed site returns. 

Look out for future parts coming soon!

-Chris