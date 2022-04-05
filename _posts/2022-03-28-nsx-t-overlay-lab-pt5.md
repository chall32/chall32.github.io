---
layout: post
title: "NSX-T 3.2: Overlay Lab Build - Part 5" 
excerpt: "Remote Tunnel Endpoints"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-01.png
comments: true
date: 2022-03-28T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-01.png">
In this post we will set up our Remote Tunnel Endpoints (RTEPs) to allow us to tunnel our overlay traffic across sites.

This post is part 5 of a multipart series.  Find the other parts here:

- Part 1: [Lab Setup and Overview](/nsx-t-overlay-lab-pt1/){:target="_blank"}
- Part 2: [Site A Build](/nsx-t-overlay-lab-pt2/){:target="_blank"}
- Part 3: [Automated Site B Build](/nsx-t-overlay-lab-pt3/){:target="_blank"}
- Part 4: [NSX-T Site Federation](/nsx-t-overlay-lab-pt4/){:target="_blank"}
- Part 5: This Part: Remote Tunnel Endpoints
- Part 6: [Federated Tier-0 Gateway](/nsx-t-overlay-lab-pt6/){:target="_blank"}

As a reminder, in this series we will be building the following lab:

<a href="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-02.png"><img style="display:block;" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-02.png" alt="NSX-T Test Lab"/></a><sup>(Click image to zoom in)</sup>

{% include _toc.html %} <br>
## What is a Remote Tunnel End Point or RTEP? 
Just like the host and edge TEPs, NSX-T Geneve traffic needs to be encapsulated and de-encapsulated by a Tunnel End Point (TEP). RTEPs are used for cross site traffic from Edge node to Edge node. If we want to pass encapsulated overlay traffic from one site to another site, we are going to need some RTEPs.

OK, so let's get some RTEPs configured!

## Check MTU
As discussed in the VMware article [NSX-T Network Requirements and Sizing for NSX-T Workload Domains](https://docs.vmware.com/en/VMware-Validated-Design/5.1/sddc-architecture-and-design-for-vmware-nsxt-workload-domains/GUID-3FF2471C-665B-4E84-8DE4-ED3F35A58DE8.html){:target="_blank"}:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="MTU Requirement" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-01.png">

Yep, we need to have a cross site maximum transmission unit (MTU) of **at least 1600**.  So to save heart ache further down the road, let's double check our site MTU settings at each site.  Let's also test our cross site MTU.

Open NSX-T Global Manager, select a site and navigate to **System > Fabric > Settings**. Confirm that both Tunnel End Point (TEP) and Remote Tunnel End Point (RTEP) settings are set to 1700 and click **Check Now**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX-T MTU Check" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-02.png">

If all OK, the overall MTU status should return "Consistent". If not, make necessary adjustments and check again.  Repeat for all other sites.

Next let's check the cross site network MTU.  Obviously the amount and complexity of testing will very much depend on the complexity of network between your NSX-T between sites.  Luckily for us, as this is a lab and as you can see from the diagram above we have just the one device between our NSX-T sites; LABROUTER.

As we are using an OPNsense for our LABROUTER, confirmation of the RTEP VLAN MTU configuration is easy. Log in to OPNsense, select **Interfaces > Diagnostics > Netstat** and lets look at our Site A RTEP VLAN interface, SITE_A_RMOTE_TRANSPORT_VL13 [As defined in Part 1](/nsx-t-overlay-lab-pt1/#site-a-vlans-and-subnets){:target="_blank"}:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Labrouter Site A RTEP Interface" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-03.png">

Looks good.  Repeat for Site B RTEP VLAN interface SITE_B_RMOTE_TRANSPORT_VL23.

## RTEP IP Pools
Next let's create some Remote Tunnel End Point IP pools.  

Open NSX-T Global Manager, select a site, then select **System > Networking > IP Address Pools > Add IP Address Pool**.

Name the Pool **Site-A-RTEP-Pool**, click **Set > Add Subnet > IP Ranges**. 

As per [Site A IP Allocation](/nsx-t-overlay-lab-pt1/#site-a-ip-allocation){:target="_blank"}, set the IP range to **192.168.13.2-192.168.13.254**, the CIDR to **192.168.13.0/24**, the Gateway IP to **192.168.13.1** and click **Add**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A RTEP Subnet" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-04.png">

Click **Apply** and **Save**. When complete you should have the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A IP Pools" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-05.png">

Again, repeat for Site B.

## Configure Inter-Location Communication
With that all done, lets enable come inter-site comms.

Open NSX-T Global Manager, select the Global Manager, then select **Location Manager**.  From there select a location and click **Networking**.

Confirm that the correct edge cluster is selected and click **Configure**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Edge Cluster" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-06.png">

Select the edge cluster again and complete the RTEP Configuration:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Cluster RTEP Config" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-07.png">

Click **Save** to complete.  Repeat for Site B.

## Configuration Confirmation
We will build our Tier 0 and Tier 1 stretched gateways in part 6. Until then, let's confirm that we are ready for them.

Open NSX-T Global Manager and select **System > System Overview**. Scroll down to the locations:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Locations" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-08.png">

Lets look closer at our RTEP "unknown" status:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="RTEP Status" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-09.png">

So our RTEP status looks good. Furthermore, clicking the *i* tells us that we are indeed ready:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="RTEPs Ready" src="/images/nsx-t-overlay-lab-pt5/nsx-t-overlay-lab-pt5-10.png">

## Conclusion and Wrap Up

So there we have it.  Our federated NSX-T sites now have their remote tunnel endpoints. We still have to create our Global Tier 0 and Tier 1 Logical routers before we can hook any VMs into our NSX-T build. We will look at that in a later part of this series.

This was part 5 of a multipart series. Find the other parts here:

- Part 1: [Lab Setup and Overview](/nsx-t-overlay-lab-pt1/){:target="_blank"}
- Part 2: [Site A Build](/nsx-t-overlay-lab-pt2/){:target="_blank"}
- Part 3: [Automated Site B Build](/nsx-t-overlay-lab-pt3/){:target="_blank"}
- Part 4: [NSX-T Site Federation](/nsx-t-overlay-lab-pt4/){:target="_blank"}
- Part 5: This Part: Remote Tunnel Endpoints
- Part 6: [Federated Tier-0 Gateway](/nsx-t-overlay-lab-pt6/){:target="_blank"}

Look out for future parts coming soon!

-Chris
