---
layout: post
title: "Simple Network Throughput Testing" 
excerpt: "Multi platform, muy sencillo!"
tags: 
- Addictive
- Android
- iPhone
- Free
- Fun
- Pro-Tip
- Speed
- Windows
image:
  thumb: network-throughput-testing/network-throughput-testing00.png
comments: true
date: 2020-02-07T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="iPerf speed" src="/images/network-throughput-testing/network-throughput-testing00.png">
There are times in tech when you get to hear about a particular piece of software written to do a particular job and you think "yeah I've heard about that, I should check it out."
<br>
<br>
Then you forget all about it (well I do) and life moves on. Months / years / decades (yeah... I know...) later, you hear about the same piece of software and once again you think "oh yeah, I really should check that out."

For me such a piece of software is iPerf.

Embarrassingly, I first heard of iPerf back in the VMware ESX 2.5 / 3.0.x days; [circa 2005](https://www.vmware.com/support/esx25/doc/releasenotes_esx25.html). Some three years before I started [posting to inter-webs](https://polarclouds.co.uk/hello/hello/).
Well, here we are, some 15 years later and I've finally got around to looking at iPerf and how to apply it to some simple network testing.

So here we are.
{% include _toc.html %}
## What is iPerf?
[Wikipedia describes iPerf](https://en.wikipedia.org/wiki/Iperf) as:
> A widely used tool for network performance measurement and tuning. It is significant as a cross-platform tool that can produce standardized performance measurements for any network. iPerf has client and server functionality, and can create data streams to measure the throughput between the two ends in one or both directions. Typical iPerf output contains a time-stamped report of the amount of data transferred and the throughput measured.

With that in mind, lets get testing.

## Downloading iPerf
Unsurprisingly given it's age, there are a few versions of iPerf to choose from.  For simplicity, we will concentrate here on the latest version of iPerf; version 3.x onwards.

The home of iPerf is [iperf.fr](https://iperf.fr/). As can be seen from the [download page](https://iperf.fr/iperf-download.php), iPerf is available for lots of platforms including Windows, Linux, FreeBSD, Apple macOS, Android and Apple iOS.

In this post, I'll be concentrating on the following distributions (click to grab your own copy):
<table>
<tr>
<td style="height:33%; width:5%;">
</td>
<td style="height:33%;  width:25%;">
<a href='https://play.google.com/store/apps/details?id=com.nextdoordeveloper.miperf.miperf'><img alt='Get it on Google Play' src='/images/fossil-collider-hr/play.png'/></a>
<i>Magic iPerf for Android</i></td>
<td style="height:33%; width:5%;">
</td>
<td style="height:33%; width:25%;">
<a href='https://iperf.fr/iperf-download.php#windows'><img alt='Windows Download' src='/images/network-throughput-testing/network-throughput-testing01.png'/></a>
<i>iPerf for Windows 64bit</i></td>
<td style="height:33%; width:5%;">
</td>
<td style="height:33%; width:25%;">
<a href="https://apps.apple.com/gb/app/iperf-3-wifi-speed-test/id1462260546"><img alt='Get it on Apple App Store' src='/images/fossil-collider-hr/appstore.png'/></a>
<i>iPerf 3 Wifi Speed Test</i></td>
<td style="height:33%; width:5%;">
</td>
</tr>
</table>
## iPerf Basics

Network testing with iPerf requires two instances of iPerf to be running at any one time:

- One instance running in server mode
- One instance running in client mode

In fact, iPerf is so simple to run to achieve basic throughput testing, the basic command set can be seen in the following diagrams:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Client Server Forward" src="/images/network-throughput-testing/network-throughput-testing03.png">

Reverse client to server testing is achieved using the **-R** switch:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Client Server Reverse" src="/images/network-throughput-testing/network-throughput-testing04.png">

## Results
### Windows to Windows
Testing from VM to VM within VMware ESXi, I achieved the following:
<table>
<tr>
<td style="height:50%;  width:50%;">
Client:<a target="_blank" href="/images/network-throughput-testing/network-throughput-testing06.png"><img style="display:block;" src="/images/network-throughput-testing/network-throughput-testing06.png" alt="VM to VM iPerf Client"/></a><sup>(Click image to zoom in)</sup>
</td>
<td style="height:50%;  width:50%;">
Server:<a target="_blank" href="/images/network-throughput-testing/network-throughput-testing05.png"><img style="display:block;" src="/images/network-throughput-testing/network-throughput-testing05.png" alt="VM to VM iPerf Server"/></a><sup>(Click image to zoom in)</sup>
</td>
</tr>
</table>
3.49Gbits per second.  Not bad :sunglasses:
### Google Pixel to Windows
Testing from my Google Pixel via WiFi to my laptop connected via a network cable to my router, I achieved the following:
<table>
<tr>
<td style="height:50%;  width:50%;">
Client:<a target="_blank" href="/images/network-throughput-testing/network-throughput-testing07.png"><img style="display:block;" src="/images/network-throughput-testing/network-throughput-testing07.png" alt="Pixel to VM iPerf Client"/></a><sup>(Click image to zoom in)</sup>
</td>
<td style="height:50%;  width:50%;">
Server:<a target="_blank" href="/images/network-throughput-testing/network-throughput-testing08.png"><img style="display:block;" src="/images/network-throughput-testing/network-throughput-testing08.png" alt="Pixel to VM iPerf Server"/></a><sup>(Click image to zoom in)</sup>
</td>
</tr>
</table>
304Mbits per second.  Nice!
### Apple iPhone to Windows
Testing from Apple iPhone via WiFi to my laptop connected via a network cable to my router, I achieved the following:
<table>
<tr>
<td style="height:50%;  width:50%;">
Client:<a target="_blank" href="/images/network-throughput-testing/network-throughput-testing09.png"><img style="display:block;" src="/images/network-throughput-testing/network-throughput-testing09.png" alt="iPhone to VM iPerf Client"/></a><sup>(Click image to zoom in)</sup>
</td>
<td style="height:50%;  width:50%;">
Server:<a target="_blank" href="/images/network-throughput-testing/network-throughput-testing10.png"><img style="display:block;" src="/images/network-throughput-testing/network-throughput-testing10.png" alt="iPhone to VM iPerf Server"/></a><sup>(Click image to zoom in)</sup>
</td>
</tr>
</table>
251Mbits per second.  Not quite as fast as the Pixel, bit still quite respectable
### Apple iPhone to Google Pixel
Finally, testing from Apple iPhone to Google Pixel via WiFi, I achieved the following:
<table>
<tr>
<td style="height:50%;  width:50%;">
Client:<a target="_blank" href="/images/network-throughput-testing/network-throughput-testing11.png"><img style="display:block;" src="/images/network-throughput-testing/network-throughput-testing11.png" alt="iPhone to Pixel Client"/></a><sup>(Click image to zoom in)</sup>
</td>
<td style="height:50%;  width:50%;">
Server:<a target="_blank" href="/images/network-throughput-testing/network-throughput-testing12.png"><img style="display:block;" src="/images/network-throughput-testing/network-throughput-testing12.png" alt="iPhone to Pixel Server"/></a><sup>(Click image to zoom in)</sup>
</td>
</tr>
</table>
WiFi to WiFi? No problem!<br>
Bit slower at 112Mbits per second, still respectable
## Conclusion
So there we have it. Some simple network throughput testing using freeware apps on common platforms.

Good to see the Pixel beat the iPhone :smile:

-Chris