---
layout: post
title: "NSX-T 3.2: Overlay Lab Build - Part 1" 
excerpt: "Lab Setup and Overview"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-01.png
comments: true
date: 2022-02-28T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-01.png">
Here we go then, eyes down for our second look at NSX-T.

If you missed the first look, have a read now: [NSX-T 3.2: Micro-Segmentation Only Deployment - Manual Setup](/nsx-t-3-2-manual-microsegmentation/){:target="_blank"} Its a cracking read honest! :wink:

In this series of posts, we'll be looking at deploying NSX-T 3.2 into a lab environment. Within the lab we will be emulating a two site enterprise with active services running from either site. On to of that, we will configure stretched layer 2 networking that will allow for us to host and failover services from one site to the other without the need for reconfiguration of those services.

This post is part 1 of a multipart series.  Find the other parts here:

- Part 1: This part - Lab Setup and Overview
- Part 2: [Site A Build](/nsx-t-overlay-lab-pt2/){:target="_blank"}
- Part 3: [Automated Site B Build](/nsx-t-overlay-lab-pt3/){:target="_blank"}
- Part 4: [Multi Site Federation](/nsx-t-overlay-lab-pt4/){:target="_blank"}
- Part 5: [Remote Tunnel Endpoints](/nsx-t-overlay-lab-pt5/){:target="_blank"}
- Part 6: [Federated Tier-0 Gateway](/nsx-t-overlay-lab-pt6/){:target="_blank"}

Right then, lets get on with part 1 of the series and take a look at the lab setup. 
{% include _toc.html %}
## The NSX-T Dual Site Lab Diagram
In this series we will be creating following lab:
<a href="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-02.png"><img style="display:block;" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-02.png" alt="NSX-T Test Lab"/></a><sup>(Click image to zoom in)</sup>

A dual site setup consisting of (imaginatively named) sites "SITE A" and "SITE B". The grey dotted line running down the middle of the diagram represents the demarcation between the sites.

## Lab Components
Lets work our way around the lab to understand the layout.  Starting top left.

**NSXT Global Manager** - This is our federated NSX-T manager. This manager provides us our single point of management and control for the three NSX-T managers located in the lab (including the this Global Manager). The global manager is also used to apply networking configuration that spans both sites. Read more here: [NSX Federation](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/3.2/administration/GUID-D5B6DC79-6733-44A7-8072-50221CF2122A.html){:target="_blank"}.

**VC-SITE-A / VC-SITE-B** - vSphere 7 vCenter servers for each site. Both vCenter servers are in enhance link mode.  Read more here: [vCenter Enhanced Linked Mode](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vcenter.install.doc/GUID-4394EA1C-0800-4A6A-ADBF-D35C41868C53.html){:target="_blank"}.

**NSXT-SITE-A / NSXT-SITE-B** - The local NSX-T manager responsible for the management and control of local NSX-T resources based in their respective site.

**Wider LAN** - The rest of my home network.  Nothing to see here, move along...

**LABROUTER** - An OPNsense router the configuration of which is discussed here:  [OPNsense BGP and BFD Configuration](/opnsense-bgp-bfd-config/){:target="_blank"}.

**ESXI-SITE-A / ESXI-SITE-B** - The ESXi host running the our production VMs and NSX-T edge located in either site.

**TIER 0 GATEWAY** - A cross site stretched top level logical router that spans across both sites A and B. Read more here: [Tier-0 Logical Router](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/3.2/administration/GUID-3F163DEE-1EE6-4D80-BEBF-8D109FDB577C.html){:target="_blank"}.

**ESG-SITE-A / ESG-SITE-B** - Our (singular) NSXT-T edge. This being a lab, we only need one edge per site.  Production deployments would (SHOULD!) have more than one edge per site. Read more here: [Create an NSX Edge Transport Node](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/3.2/installation/GUID-53295329-F02F-44D7-A6E0-2E3A9FAE6CF9.html){:target="_blank"}.

**VMs** - Our production VMs connected to either of our Tier 1 gateways / logical routers.

**TIER 1 Gateway (SITE-A Preferred)**: A cross site stretched logical router that spans both sites A and B. Ingress / Egress traffic to VMs connected to this gateway is preferentially routed to run through Site A with failover to Site B should it be required. Read more here: [Tier-1 Logical Router](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/3.2/administration/GUID-DAEF8457-8363-4F33-84DA-68AA36A2DE3C.html){:target="_blank"}.

**TIER 1 Gateway (SITE-B Preferred)**: A cross site stretched logical router that spans both sites A and B. Ingress / Egress traffic to VMs connected to this gateway is preferentially routed to run through Site B with failover to Site A should it be required. Read more here: [Tier-1 Logical Router](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/3.2/administration/GUID-DAEF8457-8363-4F33-84DA-68AA36A2DE3C.html){:target="_blank"}.

## Lab Networking Configuration 
Being that the lab is a collapsed NSX-T setup - that is our edge VMs are running alongside our production VMs on our ESXi hosts - we need to make fairly extensive use of VLANs to segregate traffic to and from our NSX-T lab.

As per [NSX-T Edge TEP networking options (83743)](https://kb.vmware.com/s/article/83743){:target="_blank"}:
Edge TEP and ESXi host TEP can be configured on the same VLAN in the following configurations:
 - Edge VM TEP interface connected to a logical switch on a vDS7 with NSX-T 3.1.0 or above

The lab is being built with vSphere 7 and NSX-T 3.2. We will be sharing TEP VLAN for our hosts and our edges.
### Site A VLANs and Subnets 
The following VLANs and subnets are used on Site A:
<style>
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
tr:nth-child(even) {background-color: #f2f2f2;}
</style>
| VLAN ID | Subnet          | Use                        | OPNsense Interface    | Colour in Diagram |
|:-------:|:----------------|:---------------------------|:----------------------|:-----------------:|
| 1       | 192.168.10.0/24 | Management Traffic         | SITE_A_MGMT           | <span style="color:#FF0000">Red</span>  |
| 11      | 192.168.11.0/24 | Transport End Points (TEPS)| SITE_A_TRANSPORT_VL11 | <span style="color:#009999">Light Blue</span> |
| 12      | 192.168.12.0/24 | Uplinks                    | SITE_A_UPLINK_VL12    | <span style="color:#009999">Light Blue</span> |
| 13      | 192.168.13.0/24 | Remote Transport End Points (RTEPS) | SITE_A_RMOTE_TRANSPORT_VL13 | <span style="color:#009999">Light Blue</span> |

### Site A IP Allocation
The following IP allocations are in use on Site A:
<style>
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
tr:nth-child(even) {background-color: #f2f2f2;}
</style>
| IP Address    | VLAN ID | Use    |
|:----------------|:----:|:---------|
| 192.168.10.10   | 1 | ESXI-SITE-A Management Interface |
| 192.168.10.15   | 1 | VC-SITE-A Management Interface |
| 192.168.10.16   | 1 | NSXT-SITE-A Management Interface |
| 192.168.10.17   | 1 | VC-SITE-A Management (Cluster Virtual IP)|
| 192.168.10.19   | 1 | NSXT Global Management Interface |
| 192.168.10.22   | 1 | ESG-SITE-A Management Interface |
|192.168.11.2-254 | 11 | Site A Tunnel End Point (TEP) Pool |
| 192.168.12.2    | 12 | Global Tier 0 Uplink - Site A |
|192.168.13.2-254 | 13 | Site A Remote Tunnel End Point (RTEP) Pool |

### Site B VLANs and Subnets
The following VLANs and subnets are used on Site B:
<style>
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
tr:nth-child(even) {background-color: #f2f2f2;}
</style>
| VLAN ID | Subnet          | Use                        | OPNsense Interface    | Colour in Diagram |
|:-------:|:----------------|:---------------------------|:----------------------|:-----------------:|
| 1       | 192.168.20.0/24 | Management Traffic         | SITE_B_MGMT           | <span style="color:#3333FF">Blue</span> |
| 21      | 192.168.21.0/24 | Transport End Points (TEPS)| SITE_B_TRANSPORT_VL21 | <span style="color:#CCCC00">Dark Yellow</span> |
| 22      | 192.168.22.0/24 | Uplinks                    | SITE_B_UPLINK_VL22    | <span style="color:#CCCC00">Dark Yellow</span>  |
| 23      | 192.168.23.0/24 | Remote Transport End Points (RTEPS) | SITE_B_RMOTE_TRANSPORT_VL23 | <span style="color:#CCCC00">Dark Yellow</span>  |

### Site B IP Allocation
The following IP allocations are in use on Site A:
<style>
table, th, td {
  border: 1px solid black;
  border-collapse: collapse;
}
tr:nth-child(even) {background-color: #f2f2f2;}
</style>
| IP Address    | VLAN ID | Use    |
|:----------------|:----:|:---------|
| 192.168.20.10   | 1 | ESXI-SITE-B Management Interface |
| 192.168.20.15   | 1 | VC-SITE-B Management Interface |
| 192.168.20.16   | 1 | NSXT-SITE-B Management Interface |
| 192.168.20.17   | 1 | VC-SITE-B Management (Cluster Virtual IP)|
| 192.168.20.22   | 1 | ESG-SITE-B Management Interface |
|192.168.21.2-254 | 21 | Site B Tunnel End Point (TEP) Pool |
| 192.168.22.2    | 22 | Global Tier 0 Uplink - Site B |
|192.168.23.2-254 | 23 | Site B Remote Tunnel End Point (RTEP) Pool |

## Lab Router Configuration
Looking at interfaces as configured on LABROUTER:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OPNsense Interfaces" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-03.png">

As previously detailed, the .1 IP address of each subnets are assigned to the interfaces and VLANs configured on the lab router. 

The WAN interface is used for connectivity to the rest of my home network.

The only other in use services running on the lab router are:

- Unbound DNS: Used for name resolution within the lab
- NTP: Time synchronization
- FRR BGP and BFD: Used for receiving and sending routing info from and to the lab

Checkout [OPNsense BGP and BFD Configuration](/opnsense-bgp-bfd-config/){:target="_blank"} for further details 

## Top Level Configuration
Unsurprisingly, I'm running this whole setup as a "nested" deployment:  

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX-T vAPP" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-08.png">

Looking closer at the top level networks used by the lab, there are just the three: 

- VM-GREEN-LAN: Provides "WAN" connectivity for the lab up to the rest of my home network.
- VM-LAB-LAN1: Connectivity for all servers on Site A 
- VM-LAB-LAN2: Connectivity for all servers on Site B

VM-LAB-LAN3 and VM-LAB-LAN4 are not used in this Lab.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX-T vAPP Networks" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-09.png">

Looking closer at the Lab Distributed switch, no uplinks to the outside world:  

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Lab Networking" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-10.png">

All lab portgroups set with the following security with VLAN trunking enabled:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Lab Networking Config" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-11.png">

MTU is set to 9000:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Lab Networking Config - MTU" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-12.png">


## Lab vCenter Configuration
Lets look at the vCenter configuration. Clusters view; Nothing complicated, two linked vCenters, two hosts and a test APP VM per site:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="vCenter Clusters" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-04.png"> 

Networking overview, again, nothing complicated, one distributed switch per site:
 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Networking Overview" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-05.png">

Finally, looking closer at the configuration of the single untagged portgroup per site for management traffic. Site A:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site A mgmt portgroup" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-06.png">

Site B:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site B mgmt portgroup" src="/images/nsx-t-overlay-lab-pt1/nsx-t-overlay-lab-pt1-07.png">

## Conclusion and Wrap Up
That'll just about do it for the first post of the series. Sure, on it's own this post is a little *dry*, however as we get to building out our NSX-T lab, this post will come into its own for reference purposes down the line.

This was part 1 of a multipart series. Find the other parts here:

- Part 1: This part - Lab Setup and Overview
- Part 2: [Site A Build](/nsx-t-overlay-lab-pt2/){:target="_blank"}
- Part 3: [Automated Site B Build](/nsx-t-overlay-lab-pt3/){:target="_blank"}
- Part 4: [Multi Site Federation](/nsx-t-overlay-lab-pt4/){:target="_blank"}
- Part 5: [Remote Tunnel Endpoints](/nsx-t-overlay-lab-pt5/){:target="_blank"}
- Part 6: [Federated Tier-0 Gateway](/nsx-t-overlay-lab-pt6/){:target="_blank"}

Look out for future parts coming soon!

-Chris