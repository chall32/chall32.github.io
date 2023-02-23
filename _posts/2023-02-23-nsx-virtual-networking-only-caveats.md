---
layout: post
title: "NSX vCenter Plug-in Deployment - Virtual Networking Configuration: Caveats" 
excerpt: "Caveats to the Virtual Networking Configuration"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-00.png
comments: true
date: 2023-02-23T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-00.png">
In two of my previous posts we looked at deploying VMware NSX via the vCenter plug-in and then configuring NSX using the Virtual Networking configuration wizard. If you missed either or both of the posts, catch up here:

- [NSX vCenter Plug-in Deployment](/nsx-vcenter-plugin-deployment/){:target="_blank"}
- [NSX vCenter Plug-in Deployment - Virtual Networking Configuration](/nsx-vcenter-virtual-networking/){:target="_blank"}

In the same vane as the post [NSX vCenter Plug-in Deployment - Security Only Configuration: Caveats](/nsx-vcenter-security-only-caveats/){:target="_blank"}, in this post we will look at the caveats that apply to building an environment using NSX Virtual Networking configuration wizard via the vCenter plug-in.

{% include _toc.html %}
## Separate Host Overlay and Edge Overlay VLANs
As we encountered during [Edge Deployment](/nsx-vcenter-virtual-networking/#edge-deployment){:target="_blank"}, the virtual networking wizard would not allow us to use the same VLAN ID for our host overlay and edge overlay connections:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edge VLAN value cannot be the same as Overlay VLAN ID" src="/images/nsx-vcenter-virtual-networking-caveats/nsx-vcenter-virtual-networking-caveats-01.png">

OK, so let's take a look at the **[VMware NSX Reference Design Guide v3.2 (pdf)](https://communities.vmware.com/t5/VMware-NSX-Documents/VMware-NSX-T-Reference-Design/ta-p/2778093){:target="_blank"}**. Page 305 onwards covers our situation (emphasis mine):

> Starting with NSX version 3.1, edge and host TEPs can reside on the same VLAN because the host now can process Geneve traffic internal to the host itself. We must transport edge VM overlay traffic over an NSX Segment in this case. If the edge TEPs are connected to a vCenter managed dvpg, tunnels between the host and the edge will not come up. This design is presented in figure 7-56 below:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Figure 7-56" src="/images/nsx-vcenter-virtual-networking-caveats/nsx-vcenter-virtual-networking-caveats-02.png">

> When connecting edge VMs to an NSX prepared VDS or NVDS, please keep in mind the following recommendations:
> - Host and Edges should be part of different VLAN Transport Zones. This ensures a clear boundary between the transport segments on the host and those used for the routing peering on the edges. The edge segment traffic is transported by the host segments, configured as a trunk.
>
> - When implementing a single TEP VLAN design like in figure 7-56, the VDS trunk port groups transporting the edge TEP traffic must be NSX managed segments and cannot be created in vCenter.
>
> - Follow the canonical recommendations regarding VLAN trunking and teaming policy configuration for overlay and VLAN peering traffic described in section: 7.5.2.2.
>
> The design with different VLAN/IP subnets per TEP is still valid and can be used with any NSX version, including 3.1 or later. **In most cases, the single TEP design is preferred for its simplicity, however for most deployments having separate VLANs for Edge and Host TEP is recommended** due to following considerations:
> - When Edges and hosts share the same TEP VLAN, they also share the span of that VLAN. While it is usually desirable to limit the host TEP VLAN to a rack, edge VMs may require mobility across racks or even sites (e.g., in the VCF stretched cluster design). Separate VLANs allow to manage the span of host and edge TEP networks individually.
>
> - An edge and the host where the edge is running will never lose TEP connectivity if they share the same TEP network, regardless of a pNIC failure. This means that the edge node VM will never incur in an all tunnels down HA condition, limiting its ability to react to specific failures. Please refer to the EDGE HA SECTION IN CHAPTER 4 for more information. (Note: a design that matches figure 7-56 should not incur any issue as management,
overlay, and VLAN peering networks share the same pNICs).

Whilst single and dual TEP VLANs are supported, it is recommended to separate host and edge TEP VLANs.

Looking back over my [NSX Overlay Lab Build Series](/nsx-t-overlay-lab-pt1/){:target="_blank"}, I covered exactly that: a lab build where we were fine in moving away from the recommendations in the name of simplicity and expediency. Not only did we use a single VLAN for my host and edge TEPs, we also deployed only one NSX manger VM and we also deployed just one edge VM. 

Shocker! :wink:<br>

The deviations from the recommendations did not harm lab functionality in any way. A lab is a playground to test, learn and break things in - it is not intended for production.

## Rebuildability
We saw in the [Security Only Configuration: Caveats](/nsx-vcenter-security-only-caveats/){:target="_blank"} post that deploying NSX via the security only wizard caused issues should the NSX build need to be extended in the future. 

I'm happy to report that no such issues exist when using the virtual networking wizard. Transport node profiles are user editable and therefore not susceptible to the same kind of issues seen previously.

## Conclusion and Wrap Up
Yep, that's better! The NSX virtual networking wizard build results in an NSX build that can be modified at a later date.

Sure it would be good to have the wizard offer options as per [NSX-T Edge TEP networking options (83743)](https://kb.vmware.com/s/article/83743){:target="_blank"}, but if the NSX build the wizard produces follows the VMware guidelines, then hey I'm good with that.

-Chris