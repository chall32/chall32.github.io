---
layout: post
title: "NSX-T 3.2: Overlay Lab Build - Part 2" 
excerpt: "Site A Build"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-01.png
comments: true
date: 2022-03-07T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-01.png">
In this post we will configure our first NSX-T site, the imaginatively named, Site A. 

This is where the "rubber meets the road". In this post not only will we deploy an NSX-T manager appliance, we will hook it into vSphere and complete the configuration required to prepare the site so that it can be 'paired' with Site B in preparation to run stretched layer 2 networks across both sites Site A and Site B.

This post is part 2 of a multipart series.  Find the other parts here:

- Part 1: [Lab Setup and Overview](/nsx-t-overlay-lab-pt1/){:target="_blank"}
- Part 2: This Part: Site A Build
- Part 3: [Automated Site B Build](/nsx-t-overlay-lab-pt3/){:target="_blank"}
- Part 4: [Multi Site Federation](/nsx-t-overlay-lab-pt4/){:target="_blank"}

As a reminder, in this series we will be building the following lab:

<a href="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-02.png"><img style="display:block;" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-02.png" alt="NSX-T Test Lab"/></a><sup>(Click image to zoom in)</sup>

{% include _toc.html %} 	
## Site A Build
### OVA Deployment + Licencing
See [NSX-T Download](/nsx-t-3-2-manual-microsegmentation/#nsx-t-download){:target="_blank"} to get your very own copy of NSX-T and an evaluation licence too!

For brevity I'm not going to cover the deployment of the NSX-T manager OVA here, suffice to say that the following options should be selected when deploying the NSX-T OVA:

- VM Name = NSXT-SITE-A
- VM Size = Small
- Hostname = nsxt-site-a
- Role = NSX Manager
- IP = 192.168.10.16
- Mask = 255.255.255.0
- Gateway / DNS / NTP = 192.168.10.1
- Enable SSH + SSH root login = ticked 

See [Site A IP Allocation](/nsx-t-overlay-lab-pt1/#site-a-ip-allocation){:target="_blank"}

### Site A Transport Zones
See [Transport Zone](/nsx-t-3-2-manual-microsegmentation/#transport-zone){:target="_blank"} for further details.

After logging into NSX-T manager, select **System > Fabric > Transport Zones > Add Zone** and create two zones, one overlay zone named **Site-A-Overlay-Transport-Zone**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Overlay TZ" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-01.png">

And one VLAN transport zone named **Site-A-VLAN-Transport-Zone**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A VLAN TZ" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-02.png">

When complete you should have the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A TZs" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-03.png">

### Site A Host and Edge Uplink Profiles
Next, lets create our uplink profiles.  See [Uplink Profile](/nsx-t-3-2-manual-microsegmentation/#uplink-profile){:target="_blank"} for further details.

Select **System > Fabric > Profiles > Uplink Profiles > Add Profile**.

Name the profile **Site-A-Host-Uplink-Profile** and scroll down to Teamings. Leave the teaming policy as **Failover Order** and name the Active Uplinks **Uplink-1,Uplink-2**. As per [Site A VLANs and Subnets](/nsx-t-overlay-lab-pt1/#site-a-vlans-and-subnets){:target="_blank"}, set the Transport VLAN to **11**. As we are using a VDS, there is no need to set an MTU:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Host Uplink Profile" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-04.png">

Select **Add Profile** again and lets create a profile named **Site-A-Edge-Uplink-Profile**.

Scroll down to Teamings. Set the teaming policy to **Load Balance Source** and name the Active Uplinks **Uplink-1,Uplink-2**. As per [Site A VLANs and Subnets](/nsx-t-overlay-lab-pt1/#site-a-vlans-and-subnets){:target="_blank"}, set the Transport VLAN to **11**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Edge Uplink Profile" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-05.png">

When complete you should have the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Uplink Profiles" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-06.png">

### Site A TEP Pool
Next, lets create our Tunnel End Point (TEP) pool. As per [NSX-T Edge TEP networking options (83743)](https://kb.vmware.com/s/article/83743){:target="_blank"} we will create a single TEP pool for use by both our edges and hosts.

A Tunnel End Point is the IP address of a transport node (Edge node or Host) used for Geneve encapsulation within a location.

Select **System > Networking > IP Address Pools > Add IP Address Pool**.

Name the Pool **Site-A-TEP-Pool**, click **Set > Add Subnet > IP Ranges**. 

As per [Site A IP Allocation](/nsx-t-overlay-lab-pt1/#site-a-ip-allocation){:target="_blank"}, set the IP range to **192.168.11.2-192.168.11.254**, the CIDR to **192.168.11.0/24**, the Gateway IP to **192.168.11.1** and click **Add**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A TEP Subnet" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-07.png">

Click **Apply** and **Save**. When complete you should have the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A TEP Pool" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-08.png">

### Attach vCenter
Next, lets attach our Site A vCenter.

Select **System > Compute Managers > Add Compute Manager**, complete the wizard, click **Add** and accept the thumbprint when prompted:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A vCenter 1" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-09.png">
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A vCenter 2" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-10.png">

When complete you should have the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A vCenter" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-11.png">

### Create Transport Node Profile
Next, lets create our Transport Node profile.  See [Transport Node Profile](/nsx-t-3-2-manual-microsegmentation/#transport-node-profile){:target="_blank"} for further details.

Select **System > Fabric > Profiles > Transport Node Profiles > Add Profile**.

Name the profile **Site-A-Transport-Node-Profile**. 

Select **VDS** and **Standard**.

Select **VC-SITE-A** and **SITE-A-DSWITCH**.

Add both **Site-A-Overlay-Transport-Zone** and **Site-A-VLAN-Transport-Zone** transport zones

Select **Site-A-Host-Uplink-Profile**

Select **Use IP Pool** and **Site-A-TEP-Pool**

Finally, select **Uplink1** and **Uplink 2**

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Trans Profile 1" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-12.png">
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Trans Profile 2" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-13.png">

When complete you should have the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Trans Profiles" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-14.png">

### Prepare Host
Next we need to apply our configuration to our compute node cluster.

Select **System > Fabric > Nodes**. In the drop down, select **VC-SITE-A**.

Next select **SITE-A-CLUSTER** and **Configure NSX**.

Select **Site-A-Transport-Node-Profile** and click Apply:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Install" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-15.png">

Allow time for the host preparation to complete:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Install in Progress" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-16.png">

### Check TEP Connectivity
Make a note of the Host's assigned TEP IP address:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Find host TEP IP" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-17.png">

Open a SSH connect to the lab router and lets see if we can ping the Host TEP IP over VLAN 11:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Ping Host TEP IP" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-18.png">

Yep, looks good.

### Create Trunk VLAN Segment
So that we may also put our Edge TEPs onto VLAN 11, we need to create a VLAN Trunk segment within NSX-T. 

Select **System > Networking > Segments > Add Segment**.

Name the Segment **Site-A-Trunk**, Connected Gateway to **None**, Transport Zone to **Site-A-VLAN-Transport-Zone** and enter VLAN of **0-4094**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Trunk" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-19.png">

Click **Save** when complete and **No** to continuing configuration.  When complete:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Segments" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-20.png">

The trunk segment should be visible in vCenter:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Trunk vCenter" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-21.png">


### Create Site A Edge Node
As this edge node is purely for our lab, lets size it accordingly.

Select **System > Fabric > Nodes > Edge Transport Nodes > Add Edge Node**. 

Name the node **ESG-SITE-A**, FQDN to **esg-site-a.lab**. Set Form Factor to **Small** 

Set CPU Reservation priority to **Normal** and Memory Reservation to **0**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 1" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-22.png">
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 2" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-23.png">

Click **Next**. Complete credentials and enable SSH logins:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 3" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-24.png">

Click **Next**. Select vCenter, Cluster and Datastore:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 4" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-25.png">

Click **Next**. As per [Site A IP Allocation](/nsx-t-overlay-lab-pt1/#site-a-ip-allocation){:target="_blank"}, assign static IP of **192.168.10.22/24** and gateway of **192.168.10.1**.

Click **Select Interface** and select **Site-A-Management**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 5" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-26.png">

Click **Save** and set DNS search domain to **lab**, DNS and NTP servers to **192.168.10.1**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 6" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-27.png">

Click **Next**. Name the switch **N-VDS-1**.

Add both **Site-A-Overlay-Transport-Zone** and **Site-A-VLAN-Transport-Zone** transport zones.

Set Uplink profile to **Site-A-Edge-Uplink-Profile**.

Select **Use IP Pool** and **Site-A-TEP-Pool**.

Finally, Set Uplink-1 and Uplink-2 to Type **VLAN Segment** and **Site-A-Trunk**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 7" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-28.png">

Click  **Save** and confirm configuration matches below:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 8" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-29.png">
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Config 9" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-30.png">

Finally, click **Finish**.

Allow time (circa 5 to 10 minutes) for the edge node to be deployed and configured:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Deploy" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-31.png">

Upon successful completion of initial configuration, the edge should have been deployed, configured and received two TEP IP addresses:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Deploy Complete" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-32.png">

Lets open an SSH connect to the lab router and lets see if we can ping the Edge TEP IPs over VLAN 11:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge TEP Ping" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-33.png">

Yep, they look good.

### Create Site A Edge Cluster
Select **System > Fabric > Nodes > Edge Clusters > Add Edge Cluster**. 

Name the cluster **Site-A-Edge-Cluster** and use the arrow to move ESG-SITE-A from the Available box to the Selected box:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge Cluster Config" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-34.png">

Click **Add**. Upon completion the following should be seen:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A Edge Cluster" src="/images/nsx-t-overlay-lab-pt2/nsx-t-overlay-lab-pt2-35.png">

## Conclusion and Wrap Up
We made it!

In this post we deployed NSX-T into and configured our first site (the imaginatively named) Site A ready to receive NSX-T federation, and some overlay configuration.

Whilst we don't yet have all the configuration in place in Site A to produce a half a working cross site NSX-T federated setup, we are well on the way. 

We still have to create our Global Tier 0 and Tier 1 Logical routers before we can hook any VMs into our NSX-T build. We will look at that in a later part of this series.

This was part 2 of a multipart series. Find the other parts here:

- Part 1: [Lab Setup and Overview](/nsx-t-overlay-lab-pt1/){:target="_blank"}
- Part 2: This Part: Site A build
- Part 3: [Automated Site B Build](/nsx-t-overlay-lab-pt3/){:target="_blank"}
- Part 4: [Multi Site Federation](/nsx-t-overlay-lab-pt4/){:target="_blank"}

Look out for future parts coming soon!

-Chris