---
layout: post
title: "VMware NSX Data Center for vSphere Failover/Failback - Part 2" 
excerpt: "Bye-bye Site A!"
tags: 
- VMware
- Pro-Tip
image:
  thumb: nsx-upgrade/nsx-upgrade-00.jpg
comments: true
date: 2020-08-05T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX" src="/images/nsx-upgrade/nsx-upgrade-00.jpg">
Last time we looked at what needs to happen should we loose an NSX Data Center primary site. We also got familiar with our NSX test lab. If you've not seen that post, [catch up now](https://polarclouds.co.uk/nsx-data-center-failover-pt1/). It's a great read. :wink: 

As mentioned, this post is part 2 of a multipart series.  Find the other parts here:

-  Part 1: [Why and Getting Familiar](https://polarclouds.co.uk/nsx-data-center-failover-pt1/)
-  Part 2: This part - Bye-bye Site A!

To recap, the NSX Data Center control plane components (consisting of the NSX Controller cluster and the Universal Logical Distributed Router (UDLR) control VMs) can only exist on one site; the primary site. In the event of loss of the primary site the control VMs must be recreated at a secondary site to reinstate the NSX control plane.
{% include _toc.html %}

In this post we will perform a controlled failover of Site A to Site B.<br>
Why controlled?  Because it includes the additional steps of preparing for the failover. Those thrust into the situation of having to recover from an unplanned failover can simply pickup the process at the [Check Site B NSX Manager Registration](https://polarclouds.co.uk/nsx-data-center-failover-pt2/#check-site-b-nsx-manager-registration) step below.

## The Lab
<a target="_blank" href="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-02.png"><img style="display:block;" src="/images/nsx-data-center-failover-pt1/nsx-data-center-failover-02.png" alt="Site A Failed"/></a><sup>(Click image to zoom in)</sup><br>
As a refresher, here is where we are trying to get to:
- NSX Controller cluster is rebuilt on Site B 
- Universal Site A UDLR control VM is rebuilt on Site B
- Universal Site B UDLR control VM is rebuilt on Site B

Additionally, Site B NSX Manager promoted to primary. The rebuilt VMs are shown in red at Site B in the above diagram.

## TL,DR - Process Overview
To Lazy, Didn't Read?<br>
Look I get it.  You are in a failure situation and need a simple overview of the steps required. You don't need a big long article to follow. You just need simple steps.  Well here you go:

1.  In case of a planned failover, shutdown the whole of Site A, including ESGs, UDLR control VMs, controller cluster and NSX Manager
2.  Check Site B NSX Manager registration
3.  Disconnect secondary NSX Manger from primary
4.  Promote Site B NSX manager to primary role
5.  Deploy new controller cluster
6.  Deploy new UDLR control VMs
7.  Verify configuration of the UDLRs
8.  Verify dynamic routing configuration of the UDLRs and ESGs
9.  Test

Breaking this down into chunks then:

### [Optional Step for Planned Failover] Shutdown Site A
First, lets double check our failover prerequisites as discussed in [here in part 1](https://polarclouds.co.uk/nsx-data-center-failover-pt1/#failover-prerequisites)

Next, lets shutdown the whole of Site-A. That includes it's Edge Service Gateway (ESG), the UDLR control VMs, controller cluster, NSX Manager and vCenter.

### Check Site B NSX Manager Registration
Access Site B NSX manager via web browser (lab: https://nsx-site-b.lab), login and navigate to **Manage vCenter Registration**.

If needed, edit (not reconfigure) **Lookup Service URL** to point to remaining vCenter (lab: vc-site-b.lab):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Lookup Service" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-01.png">

Confirm both connectivity status indicators show **Connected**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX Status Connected" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-02.png">

### Disconnect Secondary NSX Manger from Primary
Log into vCenter using the Site B URL (lab: https://vc-site-b.lab) and navigate to **Network and Security - Installation and Upgrade - Management - NSX Managers**

Select the secondary NSX manager (lab: nsx-site-b.lab). Click **Actions - Disconnect from the Primary NSX Manager** and answer **Yes** to the prompt:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Disconnect from Primary" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-03.png">

The NSX manager will now be in Transit Mode:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Transit Mode" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-04.png">

### Promote Site B NSX Manager to Primary Role
Select the NSX Manager in Transit mode (lab: nsx-site-b.lab) and Click **Actions - Assign Primary Role** and answer **Yes** to the prompt:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Transit Mode" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-05.png">

The NSX manager will now be in assigned role:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Primary" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-06.png">

### Deploy New Controller Cluster
Navigate to **Network and Security - Installation and Upgrade - Management - NSX Controller Nodes**
Click **Edit** and complete common controller attributes:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Common Controller Config" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-07.png">

Navigate to **Network and Security - Groups and Tags - IP Pools** and confirm that you have a IP pool already defined for controllers on Site B:
 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site B Controller IP Pool" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-08.png">

Back in **Network and Security - Installation and Upgrade - Management - NSX Controller Nodes**, click **Add** and complete the wizard to deploy the first controller:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Controller" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-09.png">

Once the deployment of the first controller is complete and the controller shows as Connected, repeat the process twice more to deploy two more controllers.

Once all three controllers have been deployed, confirm that they have correctly peered:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Controllers Peered" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-10.png">

### Deploy New UDLR Control VMs
Navigate to **Network and Security - NSX Edges** and select one of the previously deployed UDLRs. From there, select **Configure - Appliance Settings - Add Edge Appliance VM** and complete the wizard:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="UDLR Deployment" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-11.png">

Once deployment completes, repeat for remining UDLRs in the environment.

### Verify Configuration of the UDLRs
Navigate to **Network and Security - NSX Edges** and select one of the UDLRs. Select **Configure - Interfaces** and confirm that connectivity is as expected:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="UDLR Interfaces" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-12.png">

Confirm other settings such as firewall, DHCP Relay (if configured), etc.

Repeat verification checks on remining UDLRs in the environment.

### Verify Dynamic Routing Configuration of UDLRs and ESGs
*In my test lab, I'm using BGP for my dynamic routing. Your environment may be using OSPF so modify the following commands to fit your circumstance.*

Open a console to the Edge VM and issue the command:
{% highlight shell %}
show ip bgp neighbours summary
{% endhighlight %}
Confirm that the Edge appliance shows and "E" (Established) status with all its configured neighbouring UDLRs (lab UDLRs: 192.168.100.15 and 192.168.200.15) and the upstream router (lab LABROUTER: 192.168.222.1):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="BGP Established" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-13.png">

Next, issue the command:
{% highlight shell %}
show ip route
{% endhighlight %}
Confirm that the edge is receiving routes from both the UDLRs and the upstream router:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Routes" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-14.png">

### Test
Finally, run some trace routes to confirm that traffic is following the correct path into the environment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Trace Route In" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-15.png">

and out of the environment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Trace Route Out" src="/images/nsx-data-center-failover-pt2/nsx-data-center-failover2-16.png">

## Conclusion and Wrap Up
In this post we recovered from an outage at our primary NSX for Data Center site, Site A. With the steps detailed in this post, we were able to regain our NSX control plane and prove correct traffic ingress/egress to and from our environment.

This was part 2 of a multipart series.  Find the other parts here:

-  Part 1: [Why and Getting Familiar](https://polarclouds.co.uk/nsx-data-center-failover-pt1/)
-  Part 2: This part - Bye-bye Site A!

Next time in part 3, we'll look at what happens when Site A returns from the dead. :ghost:

*Stay tuned..!*  :smiley:

-Chris