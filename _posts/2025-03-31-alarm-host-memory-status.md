---
layout: post
title: "Quick Post: Host Memory Status Alarm"
excerpt: "Memory alarm, no memory drama"
tags: 
- Nutanix
- VMware
image:
  thumb: alarm-host-memory-status/alarm-host-memory-status-01.png
comments: true
date: 2025-03-31T00:00:00+00:00
---
A quick post along the lines of "I know how to check this when running VMware ESXi, but how do I check it when running Nutanix AHV?"

This post was prompted when I found this earlier after logging into vSphere:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Comparison" src="/images/alarm-host-memory-status/alarm-host-memory-status-02.png">

Hmm. Lets look deeper at the alarm:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Comparison" src="/images/alarm-host-memory-status/alarm-host-memory-status-03.png">


Next, let's check the hardware via the lights out card, in this case a Dell iDRAC:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Comparison" src="/images/alarm-host-memory-status/alarm-host-memory-status-04.png">

*DIMM B3: Correctable Memory Error Log Limit Reached*

OK, looks like a faulty memory module. 

The $1M question: **How do I find out more about the faulty memory module so that I can order a replacement?** Servers can be notoriously fussy when it comes to memory. 

{% include _toc.html %}
## The VMware ESXi Way 
According to Broadcom [KB-311437](https://knowledge.broadcom.com/external/article/311437/){:target="_blank"}, the command to use via an SSH session to ESXi is:
{% highlight shell %}
smbiosDump
{% endhighlight %}

Lets see:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Comparison" src="/images/alarm-host-memory-status/alarm-host-memory-status-05.png">

{% highlight shell %}
 Memory Device (Type 17): #4366
    Location: "B3"
    Manufacturer: "00CE00B300CE"
    Serial: "400CB3CF"
    Asset Tag: "00150730"
    Part Number: "M393A2G40DB0-CPB"
    Memory Array: #4096
    Form Factor: 0x09 (DIMM)
    Type: 0x1a (DDR4)
    Type Detail: 0x2080 (Synchronous, Registered)
    Data Width: 64 bits (+8 ECC bits)
    Size: 16 GB
    Max. Speed: 2133 MT/s
    Rank: 2
    Configured Speed: 2133 MT/s
    Min. Voltage: 1200 mV
    Max. Voltage: 1200 mV
    Configured Voltage: 1200 mV
{% endhighlight %}

Perfect. Putting M393A2G40DB0-CPB in to the usual suspects (ebay, Amazon) £12 later and away we go.

## The Nutanix AHV Way
So how can I get the equivalent information from Nutanix AHV?

Well, [KB-7084](https://portal.nutanix.com/kb/7084){:target="_blank"} has us covered. This time we need to open an SSH to our node's Control VM (CVM) and issue the following command:
{% highlight shell %}
ncc hardware_info show_hardware_info
{% endhighlight %}

Lets see:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Comparison" src="/images/alarm-host-memory-status/alarm-host-memory-status-06.png">

{% highlight shell %}
   | Location                      |   A3                                                             |
   | Bank connection               |   Not Specified                                                  |
   | Capable speed                 |   2133.000000 MHz                                                |
   | Current speed                 |   2133 MHz                                                       |
   | Installed size                |   32768 MB                                                       |
   | Manufacturer                  |   00CE00B300CE                                                   |
   | Product part number           |   M386A4G40DM0-CPB                                               |
   | Serial number                 |   398ECC24                                                       |
   | Type                          |   DDR4  
{% endhighlight %}

Nice!  I must have been feeling flush fitting my AHV server with 32GB modules!!

-Chris
