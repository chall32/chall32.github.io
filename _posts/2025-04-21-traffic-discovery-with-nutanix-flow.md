---
layout: post
title: "Traffic Discovery with Nutanix Flow Network Security"
excerpt: "The thrill of the chase: Uncovering hidden network traffic like a digital detective"
tags: 
- Nutanix
image:
  thumb: microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-01.png
comments: true
date: 2025-04-21T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Microsegmentation" src="/images/microsegmentation-with-nutanix-flow/microsegmentation-with-nutanix-flow-01.png">
Last time we looked at configuring [microsegmentation with Nutanix Flow Network Security (FNS)](/microsegmentation-with-nutanix-flow/){:target="_blank"}. If you have not yet had a chance to read through that post, whats stopping you? It's a great read!

As a reminder, FNS is the Nutanix software-defined microsegmentation solution integrated into Nutanix Acropolis Hypervisor (AHV), allowing for granular control over network traffic between virtual machines (VMs) and applications.

As you'll have seen from the post, we mapped out the network traffic requirements of our three tier application, implemented the required rules into FNS in enforce mode, confirmed our application functioned correctly and had a quick look at allowed and blocked traffic.

{% include _toc.html %}
## The Problem 
Whilst building FNS microsegmentation policies for known traffic flows is relatively simple, what happens if you want to use FNS for microsegmentation, but have no idea what your traffic requirements are?

Perhaps you are running a known application in a totally bespoke way, perhaps the vendor for your application is no longer around, perhaps the application in question was created in house. There are many reasons why an application may have unknown network connectivity requirements.

## Enter Flow Network Security
How can we use FNS to help in this situation?

Simple: Create an FNS monitor policy and use the discovered traffic flows to build the required security policy.

## Step 1 - Create a Traffic Discovery Category
As we touched on last time, FNS secures entities such as VMs via categories, which means our first task is to determine our category model and then "slice and dice" our application VMs into their appropriate categories. 

In this case I'm going to keep things super simple. I'll create one key called `Traffic-Discovery` and one value named after my application `Application-X`:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Categories" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-01.png">

Of course these can be named in any way you see fit.

## Step 2 - Create a Flow Monitoring Policy
Lets create a new application generic policy. I'll appropriately name the policy and I'll enable policy hit logs (at the bottom of the dialogue):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Policy 1" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-02.png">

Next, I'll add my category to secured entities (in the centre of the dialogue) and I'll remove all sources and destinations, ensuring that the policy is configured to deny all incoming and outgoing traffic:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Policy 2" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-03.png">

Finally I'll select Apply (Monitor) policy mode and confirm my monitoring policy configuration:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Policy 3" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-04.png">

As detailed, monitor mode will monitor the policy without blocking traffic.

To cut down on "log spam" hitting the syslog server, I'll ensure that my Discover-Traffic policy is the only policy with hit logs enabled:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Policy 4" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-05.png">

## Step 3 - Syslog Configuration
To be 100% sure we will be capturing security policy hit logs, lets double check Syslog configuration in Prism Central:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Syslog config 1" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-06.png">

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Syslog config 2" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-07.png">

Yep, looking good. Security policy hit logs are being forwarded to the syslog server.

## Step 4 - Add VM(s) to Traffic Discovery
We are ready to start monitoring traffic flows. Lets begin by adding our Application-X VM(s) to our traffic discovery category to begin capturing traffic flows:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add VM to Category" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-08.png">

## Step 5 - Discovering Traffic
With all set up behind us, lets see who's talking to our VM and application and perhaps more importantly who our VM and application are talking to. Traffic is shown in the interface for around 24 hours per flow.

Inbound traffic:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Inbound" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-09.png">

Outbound traffic:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Outbound" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-10.png">

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Inbound List" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-11.png">

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Outbound List" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-12.png">

Sure would be good to have an option to export the list of discovered traffic here! I've submitted an internal request to ask for one to be added. 

### Syslog
As all security policy hit logs are being sent to our syslog server, we have a full record of discovered traffic flows here as well:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Syslog" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-13.png">

Hat tip to MaxBelkov for his excellent free [Visual Syslog Server](https://maxbelkov.github.io/visualsyslog/){:target="_blank"}.

An example security policy hit log entry resembles the following:

{% highlight shell %}
10.48.52.230 Apr 10 11:12:46 local0 info
2025-04-10T10:12:51.646697+00:00 GSO-EVERGLADE01-4-1 
Genesis_1: INFO:2025/04/10 10:11:55  
[ff16a6fb-ff89-4c2d-a9c8-f67030f93d79] Discover-Traffic [Update] 
SRC=10.48.52.240 
DST=20.10.31.115 
PROTO=TCP 
SPORT=49873 
DPORT=443 
ACTION=MONITOR 
DIRECTION=OUTBOUND 
ORIG: PKTS=995 BYTES=85031 
REPLY: PKTS=585 BYTES=101450
{% endhighlight %}

Plenty to filter on there for some manual "slicing and dicing" in order to analyse and create some flow security rules.

Potentially your syslog server is more advanced allowing you to construct a log query in such a way that all required data for security policy creation is recovered in a single query and away you go from there.

### AI LLM
Having said that, manual syslog parsing is all well and good, but we aren't living in the early 2000's any more... 

{% highlight shell %}
Create a firewall rule base from the attached syslog file. 
I am only interested in traffic to/from 10.48.52.240. 
Output a table with the columns: Source, Destination, Protocol, Destination Port, Description
{% endhighlight %}

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Claude Output" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-14.png">

Thank you Claude. 

*NOTE: I'm using the free version of Claude so I've had to edit my syslog file down somewhat to meet the restrictions of the free product. Subscriptions are certainly available and you get the idea.* 

Other AI Large Language Models and implementations are available *COUGH* [Nutanix AI](https://www.nutanix.com/blog/introducing-nutanix-enterprise-ai){:target="_blank"} *COUGH*.

Yep, with some correct prompting we can get an AI LLM to create our rule base for us. Perhaps your syslog server has AI/LLM capabilities built in negating the need to use a third party?

### Security Central
Of course if you need to discover traffic at scale across multiple clusters, then [Nutanix Security Central](http://nutanix.com/ncm-sc){:target="_blank"} will absolutely be the tool for you.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSC 1" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-15.png">

Security Central is a software as a service platform used for visibility, control and optimisation of security compliance. Amongst other features, the platform ingests flow data from multiple clusters, correlating traffic information with security policies to identify potential compliance gaps without manual cross-referencing.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSC 2" src="/images/traffic-discovery-with-nutanix-flow/traffic-discovery-with-nutanix-flow-16.jpeg">

Fancy a test drive? [Go for it!](https://www.nutanix.com/products/cloud-manager/security-central-trial){:target="_blank"}

## Conclusion and Wrap Up
So there we have it. 

From here we can construct our FNS policy rules and implement them as I covered in my [Microsegmentation with Nutanix Flow Network Security](/microsegmentation-with-nutanix-flow/){:target="_blank"} post previously.

In this post we started with an unknown application and by the end of the post we had ourselves a rule base ready to be implemented into Nutanix Flow Network Security. 

Sherlock Holmes would be proud - if he did network traffic discovery... Although not much call for it in the 1880s!

-Chris