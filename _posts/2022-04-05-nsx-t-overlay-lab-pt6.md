---
layout: post
title: "NSX-T 3.2: Overlay Lab Build - Part 6" 
excerpt: "Federated Tier-0 Gateway"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-01.png
comments: true
date: 2022-04-05T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-01.png">
In this post we will create our uplink segments, deploy our Tier-0 gateway and establish our BGP connections to our lab router. 

This post is part 6 of a multipart series.  Find the other parts here:

- Part 1: [Lab Setup and Overview](/nsx-t-overlay-lab-pt1/){:target="_blank"}
- Part 2: [Site A Build](/nsx-t-overlay-lab-pt2/){:target="_blank"}
- Part 3: [Automated Site B Build](/nsx-t-overlay-lab-pt3/){:target="_blank"}
- Part 4: [NSX-T Site Federation](/nsx-t-overlay-lab-pt4/){:target="_blank"}
- Part 5: [Remote Tunnel Endpoints](/nsx-t-overlay-lab-pt5/){:target="_blank"}
- Part 6: This Part: Federated Tier-0 Gateway
- Part 7: [Federated Tier-1 Gateways](/nsx-t-overlay-lab-pt7/){:target="_blank"}
- Part 8: [Egress Traffic and MEDdling with BGP](/nsx-t-overlay-lab-pt8/){:target="_blank"}


As a reminder, in this series we will be building the following lab:

<a href="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-02.png"><img style="display:block;" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-02.png" alt="NSX-T Test Lab"/></a><sup>(Click image to zoom in)</sup>

{% include _toc.html %} <br>
## What is a Tier-0 Gateway?
A Tier-0 gateway performs the functions of a Tier-0 logical router. It processes traffic between the logical and physical networks; that is northbound traffic headed out from the NSX-T environment and southbound traffic headed in to the NSX-T environment. As the Tier-0 is federated, it is able to perform this function at both our Site A and Site B sites.

## Set Overlay Transport Zones as Default
As we are using our own transport zones way that we created back in [Part 2](/nsx-t-overlay-lab-pt2/#site-a-transport-zones){:target="_blank"} and [Part 3](/nsx-t-overlay-lab-pt3/#create-transport-zones){:target="_blank"} rather than using the pre-defined system created zones, we need to set ours as the defaults. 

Log into the Global NSX-T Manager and select **Site-A** from the task bar drop down. From there, select **System > Fabric > Transport Zones**. Select **Site-A-Overlay-Transport-Zone > Actions > Set as Default Transport Zone**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Set as Default TZ 1" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-01.png">

Click **OK** when prompted:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Set as Default TZ 2" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-02.png">

Repeat the above for Site B and Site-B-Overlay-Transport-Zone.

## Create Uplink Segments
Lets create our Tier-0 uplink segments. These will be used for north/south traffic to and from the federated gateway to the site edges.

Select the Global Manager from the task bar drop down. From there, select **Networking > Segments** then select **Add Segment**.

Name the segment **Site-A-Uplink**, ensure connected gateway is **None**.  Select Location **Site-A** and **Site-A-VLAN-Transport-Zone**. Finally, set VLAN to **12** as defined in [Part 1](/nsx-t-overlay-lab-pt1/#site-a-vlans-and-subnets){:target="_blank"}:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site-A-Uplink Segment" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-03.png">

Click **Save** and **No** to complete.  

A quick peek at Site A's vCenter networking confirms creation:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site-A-Uplink Segment vCenter" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-04.png">

Repeat for Site B, naming the uplink **Site-B-Uplink**, location as **Site-B**,  selecting **Site-B-VLAN-Transport-Zone** and setting VLAN to **22** (again as defined in [Part 1](/nsx-t-overlay-lab-pt1/#site-b-vlans-and-subnets){:target="_blank"}):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site-B-Uplink Segment" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-05.png">

Yep, looks good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site-B-Uplink Segment vCenter" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-06.png">

Back in NSX-T Global Manager, clicking **Check Status** returns **Success** for both:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Uplink Segment Status" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-07.png">

## Create Tier-0 Gateway
Lets create the federated Tier-0 gateway. Select the Global Manager from the task bar drop down. From there, select **Networking > Tier-0 Gateways**. Select **Add Tier-0 Gateway**.

Name the Gateway **Multi-Site-T0**, set the HA mode to **Active**, mark all locations as primary (i.e. both sites active rather than one active and one standby) and finally add both locations and edge clusters:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="T0 Config 1" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-08.png">

Click **Save** and **Yes** to continue the configuration of the Tier-0.

Scroll down to **Interfaces**, expand and click **Set**. 

Click **Add Interface**, name the interface **Site-A-Uplink**, location **Site-A**, IP address of **192.168.12.2/24** (again as defined in [Part 1](/nsx-t-overlay-lab-pt1/#site-a-ip-allocation){:target="_blank"}), connected to **Site-A-Uplink**, edge node **ESG-SITE-A**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="T0 Site A Uplink" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-09.png">

Click **Save**.  

Click **Add Interface** , name the interface **Site-B-Uplink**, location **Site-B**, IP address of **192.168.22.2/24** (again as defined in [Part 1](/nsx-t-overlay-lab-pt1/#site-b-ip-allocation){:target="_blank"}), connected to **Site-B-Uplink**, edge node **ESG-SITE-B**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="T0 Site B Uplink" src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-10.png">

Click **Save**. Again, click **Check Status** to confirm that the configuration is correct:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="T0 Uplinks"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-11.png">

Click **Close**. Once back in the Multi-Site-T0 configuration, confirm that both sites have one interface each:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="T0 Interfaces"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-12.png">

Next, scroll down to Route Re-distribution and click **Set** next to Site-A.

Click **Add Route Re-distribution**, Enter name of **Site-A-Route-Redistribution** and click **Set**. Select options as shown below and click **Apply**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="T0 Site A BGP"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-13.png">

Click **Add** and **Apply** to save. 

Repeat route re-distribution settings for Site B and ensure both are enabled:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site Re-Redistribution"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-14.png">

Click **Save**. Scroll back up within the configuration of Multi-Site-T0, and open the **BGP** section.

Set **Local AS** to **64605** and **Graceful Restart** to **Disable**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="T0 BGP"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-15.png">

Click **Save**.

**Set** under **BGP Neighbours** and select **Add BGP Neighbour**

Enter **192.168.12.1**, set **Location** to **Site-A**, set **BFD** **Enabled**. As per [OPNsense BGP and BFD Configuration](/opnsense-bgp-bfd-config/#configure-bgp-and-bfd) we know that our OPNsense Labrouter has a BGP AS of 64600, so add that as **Remote AS Number**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A BGP"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-16.png">

Click **Save**. Click **Check Status** to confirm BGP has established:  

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A BGP Established 1"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-17.png">

Click **i** to show further information and confirm **"Established"** status:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A BGP Established 2"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-18.png">

Click **Add BGP Neighbour** and configure for Site B location. As we are using Lab router as our site B uplink, set IP to **192.168.22.1** and remote AS as **64600** also:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site B BGP"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-19.png">

Click **Save**. Click **Check Status** to confirm BGP has established:  

<img style="display: block; margin-left: auto; margin-right: auto;" alt="All Established"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-20.png">

Again, click **i** to show further information and confirm **"Established"** status:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site B Established 2"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-21.png">

Click **Close** to close BGP Neighbours setting and **Close Editing** to close Tier-0 configuration.

Finally, click **Check Status** on the Multi-Site-T0 gateway and confirm **Success** status:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="T0 Success"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-22.png">

Nice.  And as some "icing on the cake", lets check our BGP summary in OPNsense:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OPNsense BGP"  src="/images/nsx-t-overlay-lab-pt6/nsx-t-overlay-lab-pt6-23.png">

Two established neighbours!  Perfect!

## Conclusion and Wrap Up

So there we have it.  Our Tier-0 router has been deployed and configured. BGP has been established at both sites from the Tier-0 gateway up through the edges and uplinks to our Labrouter. Our last task is to deploy two Tier-1 gateways and we will look to complete that in part 7.

This was part 6 of a multipart series. Find the other parts here:

- Part 1: [Lab Setup and Overview](/nsx-t-overlay-lab-pt1/){:target="_blank"}
- Part 2: [Site A Build](/nsx-t-overlay-lab-pt2/){:target="_blank"}
- Part 3: [Automated Site B Build](/nsx-t-overlay-lab-pt3/){:target="_blank"}
- Part 4: [NSX-T Site Federation](/nsx-t-overlay-lab-pt4/){:target="_blank"}
- Part 5: [Remote Tunnel Endpoints](/nsx-t-overlay-lab-pt5/){:target="_blank"}
- Part 6: This Part: Federated Tier-0 Gateway
- Part 7: [Federated Tier-1 Gateways](/nsx-t-overlay-lab-pt7/){:target="_blank"}
- Part 8: [Egress Traffic and MEDdling with BGP](/nsx-t-overlay-lab-pt8/){:target="_blank"}

Look out for future parts coming soon!

-Chris