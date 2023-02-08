---
layout: post
title: "NSX vCenter Plug-in Deployment - Security Only Configuration: Caveats" 
excerpt: "Caveats to Just the Software Defined Network Security"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-00.png
comments: true
date: 2023-02-08T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/nsx-vcenter-plugin-deployment/nsx-vcenter-plugin-deployment-00.png">
Over the course of my previous two posts, we have looked at deploying VMware NSX via the vCenter plug-in and then configuring the plug-in using the Security Only configuration. If you missed either or both of the posts, catch up here:

- [NSX vCenter Plug-in Deployment](/nsx-vcenter-plugin-deployment/){:target="_blank"}
- [NSX vCenter Plug-in Deployment - Security Only Configuration](/nsx-vcenter-security-only/){:target="_blank"}

In this post we will look at the caveats that apply to using NSX security only configuration via the vCenter plug-in.

{% include _toc.html %}
## The Scenario
PolarCloudsCo deployed NSX-T or [NSX as it is now known as](https://blogs.vmware.com/partnernews/2022/04/nsx-data-center-name-change.html){:target="_blank"} in a security only configuration just over twelve months ago. 

Given the success of PolarCloudsCo, they are now looking to expand their datacenter provision by adding a second site or cloud. The second site will be used for provisioning some production services as well as providing failover and resiliency for their existing services run from their existing site.

To achieve this, PolarCloudsCo would like to extend their existing NSX deployment into their second site or cloud and leverage NSX networking and routing as well as NSX security.

## The Problem
When NSX is deployed using the vSphere vCenter plug-in security only wizard option, **there is no quick way to add virtual networking to the deployment.**

To fully understand the issue, let's run through the workflow to see where the wheels fall off.

### Create Overlay Transport Zone
To enable networking and routing, we need to create an overlay transport zone. Let's create that then:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Overlay TZ" src="/images/nsx-vcenter-security-only-caveats/nsx-vcenter-security-only-caveats-01.png">

### Modify Transport Node Profile
OK, now that we have our Overlay Transport zone, we need to add it to the transport node profile that our cluster / hosts are currently attached to:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Modify Transport Node Profile" src="/images/nsx-vcenter-security-only-caveats/nsx-vcenter-security-only-caveats-02.png">

Ah.... Edit is greyed out meaning we are unable to modify it.

### Create New Transport Node Profile
OK, so let's create a brand new transport node profile, using our existing VLAN transport zone and our new Overlay transport zone:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Trans Node Profile" src="/images/nsx-vcenter-security-only-caveats/nsx-vcenter-security-only-caveats-03.png">

This results in the error:
> Using system created TZ for transportNodeProfile (Error code: 26903)

Our VLAN transport zone is system owned, therefore we are unable to modify or use it in our new transport zone profile.

### Create New VLAN Transport Zone
OK, so lets create a new VLAN transport zone as well then:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add VLAN Transport Zone" src="/images/nsx-vcenter-security-only-caveats/nsx-vcenter-security-only-caveats-04.png">

### Create New Transport Node Profile - Take 2
Let's have a second go at a new transport node profile, using our new VLAN and Overlay transport zones:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Replacement Trans Node Profile" src="/images/nsx-vcenter-security-only-caveats/nsx-vcenter-security-only-caveats-05.png">

OK, we have our existing and replacement transport node profiles.

### Apply New Transport Node Profile to Cluster
Finally, lets update our cluster to use the new Trans Node Profile:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Attach Replmnt Trans Node Profile" src="/images/nsx-vcenter-security-only-caveats/nsx-vcenter-security-only-caveats-06.png">

Fail:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Attach Replmnt Trans Node Profile Fail" src="/images/nsx-vcenter-security-only-caveats/nsx-vcenter-security-only-caveats-07.png">

> Transport Node Collection is configured for security. Update is not allowed. To update the Transport Node Collection, uninstall NSX from the cluster and then apply the desired Transport Node Profile (Error code: 26908)

Uninstalling NSX will also remove the distributed firewall and all our carefully hand crafted firewall rules. 

Without the distributed firewall, all VMs would be exposed to undesirable traffic.

So we are stuck. 

## Possible Workaround
A potential way around this would be to remove a host form the existing cluster, rebuild and use this rebuilt host as a basis to create a new cluster with its own distributed switch. From there VMs could be migrated from existing hosts and VMs into the new cluster. Existing firewall rules would follow the VMs to the new cluster.

However this would assume that there is enough capacity in the existing cluster allow the removal and rebuild of a host in the first place.

After all of that work is done however we would still have the original system generated VLAN Transport Zone and Transport Node Profile configurations floating around in our environment that cannot be removed as they remain system owned.

## Conclusion and Wrap Up
Take care when deploying NSX Security Only via the vSphere Plug-in. 

If there is even a remote possibility of a move to virtual networking after deployment, then by all means install the vCenter plug-in but manually configure security only as detailed in my post [NSX-T 3.2: Micro-Segmentation Only Deployment - Manual Setup](/nsx-t-3-2-manual-microsegmentation/){:target="_blank"} rather then using the configuration wizard.

Sometimes taking a shortcut today can cause undesired consequences tomorrow.

-Chris 