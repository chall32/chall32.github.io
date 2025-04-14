---
layout: post
title: "Microsegmentation with Nutanix Flow Network Security"
excerpt: "Segment, Segment, Go Away (Don't Come Back Another Day): How Nutanix Flow Makes Microsegmentation Simple"
tags: 
- Nutanix
image:
  thumb: microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-01.png
comments: true
date: 2025-04-14T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Microsegmentation" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-01.png">
Nutanix Flow Network Security is a software-defined microsegmentation solution integrated into Nutanix Acropolis Hypervisor (AHV).  It allows granular control over network traffic between virtual machines (VMs) and applications, enhancing security posture by isolating workloads and reducing the attack surface.  Flow Network Security uses policies based on VM categories, simplifying management and automation.  It provides visibility into network traffic flows, enabling administrators to identify and mitigate security threats effectively.

The solution offers advantages over traditional firewalls, as it operates at the VM level, providing more granular control and flexibility.  It integrates with other Nutanix services and third-party security solutions, providing a comprehensive security platform.  Nutanix Flow Network Security simplifies security management, improves compliance, and reduces the risk of data breaches.

{% include _toc.html %}
## A Three Tier Application
Consider the following traditional three tier web, app and database application, with users accessing only the web tier:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="3-Tier Application" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-02.png">

Users access the web server VM via HTTPS, which enables secure communication between their devices and the 3-Tier application web front end. The web server VM then makes requests to an application server VM, retrieving data results over TCP port 8443. This data is subsequently pulled from a database server VM through queries sent to MySQL over TCP port 3306.

Also in consideration is that all three servers also require access to DNS in order to be able to resolve the hostnames and addresses of each other. 

How can you secure this application using Nutanix Flow Network Security (FNS)?

## Categories
As FNS secures entities such as VMs via categories, our first task is to determine our category model and then "slice and dice" our VMs into their appropriate categories.

Nutanix Categories consist of a key:value pair, allowing you to define custom keys and values. Therefore I'll (imaginatively) name my category key after my application `3-Tier` and I'll create three tier values named after VM function: `App` for the application VM(s), `DB` for the database VM(s) and finally `Web` for the webserver VM(s):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Categories" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-03.png">

Once created, let's assign our VMs to our categories: 

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Category List" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-04.png">

## Flow Policy Creation - Logical
As we are securing both incoming and outgoing traffic from our application tiers, we need to carefully plan our traffic flows.

To help with this, I like to think in terms of "traffic on the wire", namely: **What traffic are we expecting our VMs to receive over the network "wire" and what traffic are we expecting our VMs to transmit over the network "wire"?**

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Traffic on the Wire" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-05.png">

Using the above diagram as reference, we can build the following first pass rule base:
<div>
<style scoped>
  table {
    border-collapse: collapse;
    border: none;
  }
  tr {
    border-top: 1px solid black;
  }
  tr:first-child {
    border-top: none;
  }
  td {
    border: none !important;
  }
</style>
</div>

|  Rule   |  Traffic "On The Wire" |  Service  |  TCP/IP Port + Protocol |
|  :------:  |  :------:  |  :------:  |  :------:  |
|  R1  |  Web from Users  |  HTTPS  |  443 TCP  |
|  R2  |  Web to App  |  Custom  |  8443 TCP  |
|  R3  |  App from Web  |  Custom  |  8443 TCP  |
|  R4  |  App to DB  |  MySQL |  3306 TCP  |
|  R5  |  DB from App  |  MySQL |  3306 TCP  |
|  R6  |  DB to DNS  |  DNS |  53 TCP/UDP  |
|  R7  |  Web to DNS  |  DNS |  53 TCP/UDP  |
|  R8  |  App to DNS  |  DNS |  53 TCP/UDP  |

## Flow Policy Creation - Actual
I'll create a single security policy for my 3-Tier application. I'll (imaginatively) name the policy `3-Tier` and I'll select to secure entities via an application generic policy as that fits our need here. I'll block IPv6 traffic as I'm only using IPv4 and I'll also enable policy hit logs to be sent to my Syslog server:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Security Policy 1" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-06.png">

After building out our eight rules from the logical list above using the visual editor, the completed configuration resembles the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Security Policy 2" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-07.png">

As you can see, I have the three categories configured in the centre as secured entities, with inbound traffic to each secured entity defined on the left and outbound traffic defined on the right. I've added the labels in red to help identify which connection represents which rule. 

Swapping the FNS configuration view from the visual interface to the list interface, we can identify our eight rules. 
#### Inbound Rules
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Security Policy Visual Editor" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-08.png">

#### Outbound Rules
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Inbound Rules" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-09.png">

Rule descriptions can be anything you like, however I've used the numbering and descriptions from the logical rule base above to aid in tracking the individual rules.

Save the policy in enforce mode. All traffic not defined in our policy will be blocked:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Outbound Rules" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-10.png">

## Testing
After a hat tip to [Doug Baer](https://blogs.vmware.com/hol/author/doug_baer){:target="_blank"} and his [3-Tier Demo App](https://blogs.vmware.com/hol/2023/01/3-tier-demo-app-2023-edition-base-template.html){:target="_blank"}, lets test the application and our FNS rules:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Application" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-11.png">

Looking good. Application running and I'm able to query the database via the application from the web using the name filter:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Application Query" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-12.png">

Yep, happy with that. Application and FNS rules working as expected. 

## Blocked Traffic
Now we know our known traffic is flowing as expected, what about unknown traffic? In other words, what have I forgotten to configure, or who's unexpectedly trying to access our VMs when (perhaps) they should not?

Opening the 3-Tier policy again, using the list view with the Show Discovered Traffic option enabled, we can easily see blocked traffic over the previous 24 hour period. For incoming traffic to the VMs we can see the following:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Inbound Dropped" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-13.png">

Interesting. Whilst the VM 2025-Jump is one of my VMs, another user of the lab is trying to access our VMs. Not a problem, their traffic was blocked.

Looking at where our VMs are trying to get to outside of just the allowed DNS connectivity (as configured in our rule base), that is unknown - or unthought about - outgoing traffic, lets see:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Outbound Dropped" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-14.png">

OK, lots of outgoing traffic to destinations via UDP port 123. Our VMs are trying to synchronise their time via [Network Time Protocol (NTP)](https://www.cbtnuggets.com/common-ports/what-is-port-123){:target="_blank"}. Yep, after checking the web, IP 100.16.142.51 is an NTP server belonging to [pool.ntp.org](https://www.ntppool.org/scores/100.16.142.51){:target="_blank"}.

## External Monitoring
As we enabled policy hit logs above, interrogating our syslog server we can construct our traffic flows. This can also be used to look for traffic flows older than the previous 24 hours. For example the following hit log shows allowed MySQL traffic exiting the App VM outgoing to the DB VM, as per Rule 4:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Syslog Entry Allow" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-15.png">

For completeness, in this syslog entry we can see our Web VM trying to get to an NTP server, this time at 45.83.234.123, with the traffic being blocked (ACTION=DROP):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Syslog Entry Drop" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-16.png">

Other options exist for external monitoring such as [Nutanix Security Central](https://www.nutanix.com/en_gb/products/cloud-manager/security-central){:target="_blank"}. The plan is to look at this in a later post.

## Conclusion and Wrap Up
There we have it. A primer on Nutanix Flow Network Security, demonstrating the microsegmentation of a three tier application consisting of three VMs each running their respective workloads. 

Hopefully you found this post informative and it will help you build your own microsegmented environment on Nutanix with AHV.

-Chris