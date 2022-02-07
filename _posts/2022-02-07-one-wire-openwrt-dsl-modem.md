---
layout: post
title: "One-Wire OpenWRT and DSL Modem Setup" 
excerpt: "All in One Connectivity and Statistics"
tags: 
- Pro-Tip
- Broadband
- ADSL
- VDSL
- Speed
image:
  thumb: /one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-01.png
comments: true
date: 2022-02-07T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-01.png">
As you may have noticed, I'm just a tiny bit of a [broadband nut](https://polarclouds.co.uk/pages/categories/#ADSL){:target="_blank"}. Don't worry, I wont get into fibre to the premises, fibre to the home, types of cabinets, G.998.4 or G.INP, etc, etc, etc...

Up until recently I'd been using an old [unlocked Huawei HG612](https://kitz.co.uk/routers/hg612unlock.htm){:target="_blank"} modem for my home broadband fibre to the cabinet service.  Given that I [like to keep an eye on my connection](https://polarclouds.co.uk/monitor-your-adsl-vdsl-connection/){:target="_blank"} and as you can probably guess, the old HG612 started to give up the ghost and started dropping my broadband connection at the slightest hint of a bit of noise on the line. 

Time for a new modem.<br>

Fair warning: This posts gets a bit geeky... and that's saying something for this site!

After searching for something that will work with G.INP and the Huawei cabinet that BT had seen fit to use in my local area, I settled on a [Zyxel VMG1312-B10A](https://service-provider.zyxel.com/global/en/products/dsl-cpes/vdsl/modemresidential-gateways/vmg1312-b-series){:target="_blank"}. Before you rush to the comments section to tell me that it is already end of life - I know and what's more I'm not bothered.

Why? Read on...<br>
{% include _toc.html %}
# Bridge Mode
Putting a router with a built in modem into bridge mode disables its routing, firewall and wireless functions. Essentially you have a device that functions as a simple modem with all routing is switched off.
## Why?
Splitting the modem and routing functions gets you the best of both worlds. With this setup you get:
- A compatible (in my case a Broadcom) modem with all its early life bugs fixed
- A modern router that quite possibly will not be the upmost compatible with your ADSL/VDSL service

Also, British Telecom's UK VDSL service employs [Dynamic Line Management (DLM)](https://kitz.co.uk/adsl/DLM.htm){:target="_blank"}. Rebooting your modem too may times in a given amount of time is frowned upon by the DLM - at the expense of internet connection speed... And speed is king!

In bridge mode, whilst the modem is responsible for the underlying VDSL connection, the upstream router is responsible for the "dial up" / Point-to-Point Protocol over Ethernet (PPPoE) data connection to the ISP. Which means I can reboot my router as many times as I want without incurring the wrath of the DLM; something I could not do with an all in one modem and router device.

Now that we understand bridge mode, lets look at the old and new and improved setups.
# The Old Setup
So that I could monitor my old HG612 Huawei modem using something like [DSLstats](http://dslstats.me.uk/){:target="_blank"}, the following setup had worked fine for years and years:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="HG612 Setup" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-02.png"> 

Two cabled connections from my OpenWRT router to my modem:
- One for the PPPoE connection to my internet service provider (as initiated by the router)
- One for modem statistics (internal access to the modem NOT over the PPPoE connection)

Nice. 
# The Problem
Whilst the Zyxel modem absolutely supports bridge mode, it had problems with the two wire (PPPoE plus stats) setup. After connecting both cables and rebooting the modem the router and modem combo would not re-initiate the PPPoE connection out to the ISP. I could see line stats but I could not access the internet. Alternatively, I could remove the stats cable and the immediately the PPPoE connection would come up and I had internet access again.

So we could have EITHER an internet connection OR modem statistics... BUT NOT BOTH!

# The Fix
When two wires won't work, just use one:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VMG1312-B10A Setup" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-03.png"> 

Lets look closer at how this works.

On closer inspection of the Zyxel in bridge mode the following can be seen:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VMG1312-B10A Bridge Setup" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-04.png"> 

Whilst the modem is in bridge mode it does have a internal access IP address of 192.168.2.1. Nice. 

The $1M question: Can this be used to gather modem stats? **it absolutely can!** 

Hooking a laptop up to the Zyxel:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Yay Stats!" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-05.png"> 

Side note: I'm using custom Zyxel firmware from johnson442 available via GitHub [HERE](https://github.com/johnson442/custom-zyxel-firmware){:target="_blank"} to generate these stats.
## Getting Stats from the LAN
So we have stats, but how can we get to them *without* connecting a laptop to the Zyxel every time?

We need some configuration on the OpenWRT router. Lets look at our initial OpenWRT configuration for the moment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OpenWRT Base Config" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-06.png"> 

We have a LAN connection and we have the PPPoE connection. However, neither of these connections are in the 192.168.2.x subnet.

Wouldn't it be good if we could add a third 'virtual' interface to the OpenWRT setup on the router that is on the 192.168.2.x subnet that we can then use to for modem stats?

Let's use that **Add new interface** button as seen in the above screenshot. Click and configure as follows:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add interface" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-07.png"> 

Back at the interfaces page, click **Edit** next to the DSLSTATS interface and complete as follows:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="DSLSTATS Config 1" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-08.png"> 

Click **Firewall Settings** and assign the interface to the wan firewall zone:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="DSLSTATS Config 2" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-09.png"> 

Finally we some Network Address Translation (NAT) to handle LAN traffic heading to 192.168.2.1. Back in OpenWRT, select **Network > Firewall > NAT Rules**.

Click **Add** and complete as follows:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="DSLSTATS NAT" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-10.png"> 

Source address is a device on our internal network, destination address is our modem, action is SNAT (source NAT) and finally our source address is a custom address in the same subnet as our modem, in this case 192.168.2.2.

Click **Save**. The config should resemble the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NAT Config" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-11.png">  and test from the internal device specified above:

Finally, test from the LAN device specified in the NAT rule:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Boom!" src="/images/one-wire-openwrt-dsl-modem/one-wire-openwrt-dsl-modem-12.png"> 

Boom! Modem stats and a stable PPPoE connection!

# Conclusion and Wrap Up
In this (albeit edge case) post, we were able to configure our OpenWRT router with a source NAT (SNAT) which in turn allows us to obtain VDSL modem stats over the same cable used by our router for our PPPoE internet connection. 

Happy NATing!

-Chris