---
layout: post
title: "VMware NSX Data Center for vSphere Failover/Failback - Part 3" 
excerpt: "Site A Back from the Dead!"
tags: 
- VMware
- Pro-Tip
image:
  thumb: nsx-upgrade/nsx-upgrade-00.jpg
comments: true
date: 2020-08-09T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX" src="/images/nsx-upgrade/nsx-upgrade-00.jpg">
Last time we recovered from the loss of our NSX Data Center primary site. If you've not seen that post, [catch up now](https://polarclouds.co.uk/nsx-data-center-failover-pt2/). It's a great read. :wink: 

As mentioned, this post is part 3 of a multipart series.  Find the other parts here:

-  Part 1: [Why and Getting Familiar](https://polarclouds.co.uk/nsx-data-center-failover-pt1/)
-  Part 2: [Bye-bye Site A!](https://polarclouds.co.uk/nsx-data-center-failover-pt2/)
-  Part 3: This part - Site A Back from the Dead!
-  Part 4: [Making Site A Primary Again](https://polarclouds.co.uk/nsx-data-center-failover-pt4/)

To recap, the NSX Data Center control plane components (consisting of the NSX Controller cluster and the Universal Logical Distributed Router (UDLR) control VMs) can only exist on one site; the primary site. In the event of loss of the primary site the control VMs must be recreated at a secondary site to reinstate the NSX control plane. When we lost the primary site, we recreated them at secondary site to reinstate the NSX control plane.
{% include _toc.html %}
## The Lab
<a target="_blank" href="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-02.png"><img style="display:block;" src="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-02.png" alt="Site A Failed"/></a><sup>(Click image to zoom in)</sup><br>
As a refresher, here is where we are:
- NSX Controller cluster has been rebuilt on Site B 
- Universal Site A UDLR control VM has been rebuilt on Site B
- Universal Site B UDLR control VM has been rebuilt on Site B

Additionally, Site B NSX Manager is now our primary manager.

## TL,DR - Process Overview
To Lazy, Didn't Read?<br>
Again here are the process steps for those TL,DRs among us:

1.  Start Site A
2.  Check Site A NSX Manager Configuration and Status
3.  Demote Site A NSX Manager
4.  Delete Site A Controller Cluster
5.  Delete Site A UDLR Control VMs
6.  Assign Site A NSX Manager Secondary Role
7.  Verify Dynamic Routing Configuration of UDLRs and ESGs
8.  Test

### Site A Start Up
OK, so site A is back from the dead.  Lets get the site powered back on so we can reinstate it as an NSX secondary site in the first instance. From there we can promote it to primary again. Lets power on Site A in the following order:

- ESXi Host(s)
- vCenter Server
- NSX Manager
- Controller Cluster
- Universal Logical Distributed Router (UDLR) Control VMs
- Edge Service Gateways (ESG) VMs

### Check Site-A NSX Manager Configuration and Status
Just as we did with Site B's NSX manager during failover in part 2, let's confirm that Site A's NSX Manager is happy and registered with vCenter.  Access Site A NSX manager via web browser (lab: https://nsx-site-a.lab), login and navigate to **View Summary**. Confirm that the NSX Management Components are running:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX Management Components" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-01.png">

and confirm vCenter Registration **Home - Manage vCenter Registration** shows as green:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="vCenter Registration" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-02.png">

### Demote Site A NSX Manager
Log onto Site A vCenter (lab: https://vc-site-a.lab/), navigate to **Network and Security - Installation and Upgrade - Management - NSX Managers**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Two Primary Managers" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-03.png">

As you can see, both Site A and Site B NSX Managers believe that they are the primary NSX Manager. Lets look closer at the sync issue:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Sync Issue" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-04.png">

Fair enough, we disconnected Site B NSX Manager from Site A NSX Manager during the failover.

Select Site A NSX Manager and select **Actions - Remove Secondary Manager**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remove Secondary Manager" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-05.png">

Tick select **Perform Operation even if NSX Manager is inaccessible** and **Remove**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Perform Removal" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-06.png">

Next, select Site A NSX Manager again and select **Actions - Remove Primary Role**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remove Primary Role" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-07.png">

Answer **Yes** to the warning (we'll clean up our controllers in the next step):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Yes to Warning" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-08.png">

Site A NSX Manager will then be placed into Transit mode:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Manager in Transit Mode" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-09.png">

### Delete Site A Controller Cluster
Navigate to **Network and Security - Installation and Upgrade - NSX Controller Nodes** and select the NSX Manager in Transit (lab: NSX Manager 192.168.10.4). Select each controller in turn and select **Delete**, allowing time for deletion between each:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Delete Controllers" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-10.png">

Upon deletion of the final controller, tick **Proceed to Force Delete** and click **Delete**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Force Delete" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-11.png">

### Delete Site A UDLR Control VMs
Navigate to **Network and Security - NSX Edges** and select the NSX Manager in Transit (lab: NSX Manager 192.168.10.4).
Select first UDLR VM listed and select **Delete**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Delete UDLR" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-12.png">

Confirm deletion by clicking **Delete** again.

Repeat for remaining UDLR control VMs, leaving only ESG(s) listed:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Just ESG remaining" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-13.png">

### Assign Site A NSX Manager Secondary Role
Navigate to **Network and Security - Installation and Upgrade - Management - NSX Managers**, select NSX Manager with primary role (lab: nsx-site-b.lab) and select **Actions - Add Secondary Manager**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Secondary Manager" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-14.png">

Complete wizard entering Site A NSX Manager details and click **Add**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Secondary Manager Wizard" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-15.png">

Accept thumbprint and confirm that Site A NSX Manager is now listed as a Secondary Manager:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Secondary Manager Added" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-16.png">

Navigate to back to **Network and Security - NSX Edges** select NSX Manager with the newly assigned secondary role (lab: nsx-site-a.lab) and confirm that UDLRs are again listed:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="UDLRs are back" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-17.png">

### Verify Dynamic Routing Configuration of UDLRs and ESGs
*In my test lab, I'm using BGP for my dynamic routing. Your environment may be using OSPF so modify the following commands to fit your circumstance.*

Open a console to the Site A Edge VM and issue the command:
{% highlight shell %}
show ip bgp neighbours summary
{% endhighlight %}
Confirm that the Edge appliance shows and "E" (Established) status with all its configured neighbouring UDLRs (lab UDLRs: 192.168.100.15 and 192.168.200.15) and the upstream router (lab LABROUTER: 192.168.111.1):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="BGP Established" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-18.png">

Next, issue the command:
{% highlight shell %}
show ip route
{% endhighlight %}
Confirm that the edge is receiving routes from both the UDLRs and the upstream router:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Routes" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-19.png">

### Test
Finally, run some trace routes to confirm that traffic is following the correct path into the environment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Trace Route In" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-20.png">

and out of the environment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Trace Route Out" src="/images/nsx-data-center-failover-pt3/nsx-data-center-failover3-21.png">

## Conclusion and Wrap Up
In this post we recovered our primary NSX for Data Center site, Site A. With the steps detailed in this post, we demoted our back from the dead Site A to secondary site status, regained our NSX control plane and proved correct traffic ingress/egress to and from our previously dead site.

This was part 3 of a multipart series.  Find the other parts here:

-  Part 1: [Why and Getting Familiar](https://polarclouds.co.uk/nsx-data-center-failover-pt1/)
-  Part 2: [Bye-bye Site A!](https://polarclouds.co.uk/nsx-data-center-failover-pt2/)
-  Part 3: This part - Site A Back from the Dead!
-  Part 4: [Making Site A Primary Again](https://polarclouds.co.uk/nsx-data-center-failover-pt4/)


Next time, in part 4, we'll look at promoting Site A back to primary status again.

*Stay tuned..!*  :smiley:

-Chris