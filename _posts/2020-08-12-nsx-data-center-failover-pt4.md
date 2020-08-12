---
layout: post
title: "VMware NSX Data Center for vSphere Failover/Failback - Part 4" 
excerpt: "Making Site A Primary Again"
tags: 
- VMware
- Pro-Tip
image:
  thumb: nsx-upgrade/nsx-upgrade-00.jpg
comments: true
date: 2020-08-12T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX" src="/images/nsx-upgrade/nsx-upgrade-00.jpg">
Last time we recovered from the loss of our NSX Data Center primary site. If you've not seen that post, [catch up now](https://polarclouds.co.uk/nsx-data-center-failover-pt3/). It's a great read. :wink: 

As mentioned, this post is part 4 of a multipart series.  Find the other parts here:

-  Part 1: [Why and Getting Familiar](https://polarclouds.co.uk/nsx-data-center-failover-pt1/)
-  Part 2: [Bye-bye Site A!](https://polarclouds.co.uk/nsx-data-center-failover-pt2/)
-  Part 3: [Site A Back from the Dead!](https://polarclouds.co.uk/nsx-data-center-failover-pt3/)
-  Part 4: This Part - Making Site A Primary Again

To recap, the NSX Data Center control plane components (consisting of the NSX Controller cluster and the Universal Logical Distributed Router (UDLR) control VMs) can only exist on one site; the primary site. In the event of loss of the primary site the control VMs must be recreated at a secondary site to reinstate the NSX control plane. When we lost the primary site, we recreated them at secondary site to reinstate the NSX control plane.  In this post we will promote Site A back to being our primary site.
{% include _toc.html %}
## The Lab
<a target="_blank" href="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-01.png"><img style="display:block;" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-01.png" alt="Site A Back"/></a><sup>(Click image to zoom in)</sup><br>
As a refresher, here is where we are:
- NSX Controller cluster has been rebuilt on Site B 
- Universal Site A UDLR control VM has been rebuilt on Site B
- Universal Site B UDLR control VM has been rebuilt on Site B

Additionally, Site B NSX Manager is now our primary manager.

We need to make Site A our primary site again.

## TL,DR - Process Overview
To Lazy, Didn't Read?<br>
Yep, still got you covered:

1.  Remove Primary Role and Assign to Site A NSX Manager
2.  Deploy Site A controller cluster
3.  Deploy Site A UDLR control VMs
4.  Delete Site B controller cluster
5.  Assign Secondary role to Site B NSX Manager
6.  Confirm Site A UDLR controller VM clean up
7.  Verify configuration of the UDLRs
8.  Verify dynamic routing configuration of the UDLRs and ESGs
9.  Test

### Remove Primary Role and Assign to Site A NSX Manager
Log onto Site A vCenter (lab: https://vc-site-a.lab/), navigate to **Network and Security - Installation and Upgrade - Management - NSX Managers**, select primary NSX Manager, click **Actions - Remove Primary Role**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remove Primary Role" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-02.png">

Answer **Yes** to continue.

Once complete, both NSX Managers will be placed into transit mode:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Transit Mode" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-03.png">

Select Site A NSX Manager and click **Actions - Assign Primary Role**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Assign Primary Role" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-04.png">

### Deploy Site A Controller Cluster
Navigate to **Network and Security - Installation and Upgrade - Management - NSX Controller Nodes**
Confirm that Primary (Site A) NSX Manager is selected, confirm common controller attributes and click **Add** to deploy the first Site A controller:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Controller" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-05.png">

Once the deployment of the first controller is complete and the controller shows as Connected, repeat the process twice more to deploy two more controllers.

Once all three controllers have been deployed, confirm that they have correctly peered:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Controllers Peered" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-06.png">

### Deploy Primary Site UDLR Control VMs
Navigate to **Network and Security - NSX Edges**, confirm that Primary (Site A) NSX Manager is selected and select one of the previously deployed UDLRs. From there, select **Configure - Appliance Settings - Add Edge Appliance VM** and complete the wizard:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="UDLR Deployment" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-07.png">

Once deployment completes, repeat for remining UDLRs in the environment until deployment status for all primary site Edges equals **Deployed**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Primary Edges" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-08.png">

### Delete Site B Controller Cluster
Navigate to **Network and Security - Installation and Upgrade - Management - NSX Controller Nodes** and confirm that Transit (Site B) NSX Manager is selected. Select each controller in turn and select Delete, allowing time for deletion between each:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Delete Controllers" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-09.png">

Upon deletion of the final controller, tick **Proceed to Force Delete** and click **Delete**:

### Assign Secondary Role to Site B NSX Manager
Navigate to **Network and Security - Installation and Upgrade - Management - NSX Managers**, select primary NSX Manager, click **Actions - Add Secondary Manager**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Secondary Manager" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-10.png">

Complete wizard and click **Add**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Complete Secondary Manager Wizard" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-11.png">

Accept thumbprint and confirm that Site B NSX Manager is now listed as a Secondary Manager:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Secondary Manager" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-12.png">

### Confirm Site A UDLR Controller VM Clean Up
Navigate to **Network and Security - NSX Edges**, confirm that Secondary (Site B) NSX Manager is selected and confirm status of UDLRs is listed as **Active** instead of **Deployed**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="UDLRs Active" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-13.png">

Finally, confirm that the controller VMs have been deleted from the secondary site:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Inventory" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-14.png">

### Verify Configuration of the UDLRs
Navigate to **Network and Security - NSX Edges** in the primary site and select one of the UDLRs. Select **Configure - Interfaces** and confirm that connectivity is as expected:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="UDLR Interfaces" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-15.png">

Confirm other settings such as firewall, DHCP Relay (if configured), etc.

Repeat verification checks on remining UDLRs in the environment.

### Verify Dynamic Routing Configuration of UDLRs and ESGs
*In my test lab, I'm using BGP for my dynamic routing. Your environment may be using OSPF so modify the following commands to fit your circumstance.*

Open a console to both of the Edge VMs in turn and issue the command:
{% highlight shell %}
show ip bgp neighbours summary
{% endhighlight %}
Confirm that the Site A Edge appliance shows and "E" (Established) status with all its configured neighbouring UDLRs (lab UDLRs: 192.168.100.15 and 192.168.200.15) and the upstream router (lab LABROUTER: 192.168.111.1):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESG A Established" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-16.png">

Confirm that the Site A Edge appliance shows and "E" (Established) status with all its configured neighbouring UDLRs (lab UDLRs: 192.168.100.15 and 192.168.200.15) and the upstream router (lab LABROUTER: 192.168.222.1):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESG B Established" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-17.png">

Next, issue the command:
{% highlight shell %}
show ip route
{% endhighlight %}
Confirm that the both edges are receiving routes from both the UDLRs and the upstream router:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A ESG Routes" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-18.png">

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site B ESG Routes" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-19.png">

### Test
Finally, run some trace routes to confirm that traffic is following the correct path into the environment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Traffic Ingress" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-20.png">

and out of the environment. Site A:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Traffic Egress" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-21.png">

Site B:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site B Traffic Egress" src="/images/nsx-data-center-failover-pt4/nsx-data-center-failover4-22.png">

## Conclusion and Wrap Up
So there we have it.  The complete failure of an NSX Data Center primary site, promotion of secondary site to primary site status and subsequent recovery of control plane in remaining site. Once the failed site came back online the repromotion of the site to primary status and the clean up of temporary control plane in the newly demoted site.

Phew!  That's it for this multipart series. Hope you enjoyed it. Remember to a link to this series safe. You never know when you may need it!

Find the other parts here:

-  Part 1: [Why and Getting Familiar](https://polarclouds.co.uk/nsx-data-center-failover-pt1/)
-  Part 2: [Bye-bye Site A!](https://polarclouds.co.uk/nsx-data-center-failover-pt2/)
-  Part 3: [Site A Back from the Dead!](https://polarclouds.co.uk/nsx-data-center-failover-pt3/)
-  Part 4: This Part - Making Site A Primary Again

All in all, a bit of a mission this one, but well worth it should disaster ever strike. :grimacing:

-Chris