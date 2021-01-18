---
layout: post
title: "NSX for vSphere Load Balancer Firewall Problem" 
excerpt: "Firewall not -er- Firewalling"
tags: 
- VMware
- Pro-Tip
image:
  thumb: /nsx-upgrade/nsx-upgrade-00.jpg
comments: true
date: 2021-01-18T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX" src="/images/nsx-upgrade/nsx-upgrade-00.jpg">
Super quick tip time... 

I discovered this little nugget of configuration joy today, a case where an NSX-v edge configured as a load balancer was not firewalling traffic as expected. Bit of a head scratcher to start with, however the answer is out there in the documentation... The problem was finding it!

Hopefully this post will serve as a point of reference / reminder for me should this issue come up again in the future.

## Scenario
Using an NSX load balancer to host a load balanced Virtual IP (VIP) for SMTP email relay to Microsoft Exchange. 

So not all LAN hosts can relay email to Exchange as this would be a security risk, the load balancer edge firewall was configured with an allow IP set of IP addresses approved to relay SMTP email. LAN clients not in the allow IP set should be blocked from relaying SMTP email to Exchange.

## Problem
Upon testing, it was found that SMTP traffic from ***ALL*** LAN hosts was being allowed to Exchange via the load balancer despite the firewall rule described above.

## Resolution
After much edge firewall diagnosis and spot of Googling, I found this article: [Configure Load Balancer Service]( https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.admin.doc/GUID-B45A6901-E1EA-42A6-98F1-7AEAD5BAB193.html). 

Within this article, I found these nuggets of information:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX Load Balancer Acceleration" src="/images/nsx-load-balancer-firewall-problem/nsx-load-balancer-firewall-problem-01.png">

To quote:
> When disabled, all virtual IP addresses (VIPs) use the L7 LB engine.

> The L7 HTTP/HTTPS VIPs ("acceleration disabled" or L7 setting such as AppProfile with cookie persistence or SSL-Offload) are processed after the edge firewall, and require an edge firewall allow rule to reach the VIP.

Further,

> The L4 VIP ("acceleration enabled" in the VIP configuration and no L7 setting such as AppProfile with cookie persistence or SSL-Offload) is processed before the edge firewall, and no edge firewall rule is required to reach the VIP.

As I want to use the edge load balancer firewall rule to govern the relaying of SMTP email to Exchange, clearly I need to use the L7 load balancer engine - I.E. ***Ensure acceleration is set to "Disabled"***.

Double checking my config:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Ooops!" src="/images/nsx-load-balancer-firewall-problem/nsx-load-balancer-firewall-problem-02.png">

Opps!<br>

With acceleration being enabled, the firewall rules were are not being applied to the traffic, hence all LAN hosts were able to relay email to the Exchange!

Right, simple enough, fix:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Fixed" src="/images/nsx-load-balancer-firewall-problem/nsx-load-balancer-firewall-problem-03.png">

Fixed!

Sometimes acceleration isn't needed. Sometimes slow and steady wins the race :turtle: 

Nice little fix anyway.

-Chris