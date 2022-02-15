---
layout: post
title: "OPNsense BGP and BFD Configuration" 
excerpt: "Getting ready for NSX-T: Free Routing"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-01.png
comments: true
date: 2022-02-15T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="OPNsense Logo" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-01.png">
In preparation for our next look into NSX-T overlay networking and stretched layer 2 networks, we need to take a look into configuring our NSX-T lab router. Specifically BGP and BFD. 
{% include _toc.html %}
## What is OPNsense
OPNsense is an open source, easy-to-use and easy-to-build FreeBSD based firewall and routing platform. OPNsense includes most of the features available in expensive commercial firewalls, and more in many cases. It brings the rich feature set of commercial offerings with the benefits of open and verifiable sources. Find out more here: [OPNsense.org](https://opnsense.org/){:target="_blank"}

## What is BGP?
 Border Gateway Protocol (BGP) refers to a gateway protocol that enables the internet to exchange routing information between autonomous systems (AS). As networks interact with each other, they need a way to communicate. This is accomplished through peering. BGP makes peering possible. Without it, networks would not be able to send and receive information with each other. Find out more here: [What is BGP?](https://www.cloudflare.com/en-gb/learning/security/glossary/what-is-bgp/){:target="_blank"}

## What is BFD?
Bidirectional Forwarding Detection (BFD) is a network protocol that is used to detect faults between two routers or switches connected by a link. It provides low-overhead detection of faults even on physical media that doesn't support failure detection of any kind, such as Ethernet, virtual circuits, tunnels and MPLS Label Switched Paths. Find out more here: [Understanding BFD for BGP](https://www.juniper.net/documentation/en_US/junos/topics/concept/bgp-bfd-understanding.html){:target="_blank"}

## OPNsense Lab Router 
Without giving too much away of my upcoming NSX-T lab posts, lets take a look at where the OPNsense lab router setup sits within the context of the lab:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Lab Router" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-02.png"> 

Effectively the lab router is connected between the NSX-T lab environment and my wider home network. It's primary job is to emulate a wider enterprise network that the NSX-T lab environment plugs in to. As part of that role, the lab router runs both BGP and BFD to push and receive network routing to and from the NSX-T lab.

This post details how to install and configure the FRR BGP service on OPNsense in preparation for building out the NSX-T lab.

## Installing OPNsense
I wont duplicate the [install guide](https://docs.opnsense.org/manual/install.html){:target="_blank"} other than saying that I did a full install into a VM using the ISO image, rather than an embedded install from the IMG file.

## Configuring OPNsense for an NSX-T Lab
Follows is a configuration guide to configure OPNsense for use as described above.

### Disable the Firewall
As the lab router sits between my NSX-T lab and my wider home network, there is no real need to have the OPNsense firewall operational.

Select **Firewall > Advanced > Disable Firewall**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Disable Firewall" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-03.png"> 

### Install FRR Package
FRRouting (FRR) is a free and open source Internet routing protocol suite for Linux, Unix and BSD platforms. It implements BGP and BFD amongst other protocols. Read more about it at [FRRouting.org](https://frrouting.org/){:target="_blank"}

Select **System > Firmware > Plugins**, search for "FRR" and click **+** to install:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Install FRR" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-04.png"> 

### Configure BGP and BFD
Once the FRR package is installed, refresh the browser to populate the left hand menu, select **Routing > General > Enable** and **Save**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Enable FRR" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-05.png"> 

Select **BGP** and **Enable**.  Enter your local **[BGP AS Number](https://www.thousandeyes.com/learning/glossary/as-autonomous-system){:target="_blank"}** and set Route Redistribution to **Connected routes (directly attached subnet or host)**. Click **Save** when done:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="BGP General Config" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-06.png"> 

Select the **Prefix Lists** tab and click **+**.<br>
We want to allow any [Prefix List](https://packetlife.net/blog/2010/feb/1/understanding-ip-prefix-lists/){:target="_blank"}, so configure as follows, Again click **Save** when done:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="BGP Prefix List" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-07.png"> 

Select the **Route Maps** tab and click **+**. <br>
Again, this being a lab, we are not interested in any [route filtering](https://www.interxion.com/hr/blogs/2017/07/using-the-route-maps-for-bgp-filtering){:target="_blank"}, so configure to permit as follows. Click **Save** when done:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="BGP Route Maps" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-08.png"> 

Finally, lets enable BFD. Select **Routing > BFD > Enable** and **Save**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Enable BFD" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-09.png"> 

### Pulling it Altogether
We couldn't complete all of the above without taking a quick sneak peak at how all of the above configuration is pulled together when setting up a BGP neighbour.  

In the top highlight box of the screenshot below, we see specifics relating to our BGP neighbour. We have our peer device's description, IP address, AS number and the local interface we expect to use to peer with the device.

The middle highlight box shows that we will also use BFD with this peer.

Finally, the bottom highlight box shows that we will be allowing any prefix lists with no filtering both in and out:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="BGP Neighbour" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-10.png"> 

The BFD neighbour setup is super simple:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="BFD Neighbour" src="/images/opnsense-bgp-bfd-config/opnsense-bgp-bfd-config-11.png"> 

For our upcoming NSX-T Overlay lab, we'll configure our BGP and BFD neighbours as part of the lab build.

## Conclusion and Wrap Up
In this post we looked at OPNsense, it's position in our upcoming NSX-T lab and installing the FRRouting package onto OPNsense.

Finally we configured Boarder Gateway Protocol (BGP) and Bidirectional Forwarding Detection (BFD).

-Chris 