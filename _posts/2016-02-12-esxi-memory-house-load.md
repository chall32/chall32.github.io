---
layout: post
title: VMware ESXi In House Memory Load
excerpt: How much memory does VMware ESXi consume for itself? 
tags:
- ESXi
- VMware
image:
  thumb: esxi-memory-house-load/esxiload00.png
comments: true
date: 2016-02-12T18:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="ESXi Memory Load" src="/images/esxi-memory-house-load/esxiload00.png">
What is an "in house load"?  
Well first off, it's got nothing to do with this guy:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Dr House" src="/images/esxi-memory-house-load/esxiload01.jpeg">

By way of an explanation:

***3.0 HOUSE LOAD OPERATION*** <br>
*A unit will be in house load operation when it is disconnected from the grid and feeding the power for its own auxiliaries.* <br>
From [Power Plant Instrumentation and Control Handbook](https://books.google.co.uk/books?id=Ns06BAAAQBAJ&lpg=PA900&ots=EgyayybWbH&dq=what%20is%20%22in-house%20load%22&pg=PA900#v=onepage&q=what%20is%20%22in-house%20load%22&f=false)

So extrapolating this out for VMware ESXi Server and memory; how much memory does ESXi use for itself? 

Unfortunately VMware don't seem to publish any figures for this value for the currently supported versions of ESXi available.  This is more than likely because it is very much a hardware dependant figure.  That is depending on the hardware make up of a physical server and hence the hardware drivers needed to run that hardware server "X" may consume more system memory than server "Y" running with a different hardware make up. Also, memory loading may change depending on the options you choose to run at ESXi level; for example iSCSI, NFS, etc.

{% include _toc.html %}

## Why Measure ESXi Memory In House Load?
So this is the predicament I find myself in.

I have a stand-alone ESXi instance in the CH-Datacentre<sup>TM</sup> Lab that has 6GB of memory fitted:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Spongebob" src="/images/esxi-memory-house-load/esxiload02.png">

As you can see, spongebob (yes my home lab naming scheme comes from a [certain cartoon](http://www.imdb.com/title/tt0206512/)) is quite happy running with just over 512MB memory free.  However that's running ESXi 5.1... and ESXi 5.1 goes end of general support on [24 August 2016](http://www.vmware.com/files/pdf/support/Product-Lifecycle-Matrix.pdf). 

So the question is, if I upgrade ESXi to 5.5, or even 6.0, how much of that 512MB free memory will a later version of ESXi eat into?

Sure, I could just purchase some more memory and be done with it.... Oh if it was that easy. You see the slightly elderly system board I have running ESXi seems to be rather fussy when it comes to memory DIMMs it will support. I would rather not spend hours down the rabbit hole of looking for supported DIMMs that will/will not work for potentially no reason. Surely newer versions of ESXi wont use *that* much more memory will they?

## The Test Rig
To get a level playing field, I configured a VMware Workstion 11 virtual machine with the following hardware make up:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Rig VM" src="/images/esxi-memory-house-load/esxiload03.png">

4GB RAM, 2 CPU's, 15GB HDD, 2 NICs and thats about it.  Remember we are not interested in running VMs inside the test ESXi installs here.
The test ESXi installs were installed via .iso image, rebooted and had their IP addresses set by DHCP.  No further ESXi configuration was made to them.

Sure this is probably a sketchy way of testing, and not particularly indicative of a real world example. However what we are looking for here are the differences in memory load in each version of ESXi across as level playing field as possible. Therefore our hardware config doesnt matter as long as it remains exactly the same across each test instance. 

## ESXi Versions Tested
Again, to try and get a level playing field, I've used the very latest builds of ESXi available at the time of writing. These are:

- ESXi 5.1 Build 3070626 - ESXi 5.1 Patch 8, released 01 October 2015
- ESXi 5.5 Build 3343343 - ESXi 5.5 Express Patch 9, released 04 January 2016
- ESXi 6.0 Build 3380124 - ESXi 6.0 Update 1b, released 07 January 2016

When it comes to matching VMware build numbers to patches/updates and release dates, VMware [KB1014508](http://kb.vmware.com/kb/1014508) is invaluable!

### ESXi 5.1 Memory Load
Using the above test rig and after letting the v5.1 ESXi install settle for 5 minutes, the following can be seen:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi 5.1" src="/images/esxi-memory-house-load/esxiload04.png">

**ESXi 5.1 in house memory load is circa 1052MB**

### ESXi 5.5 Memory Load
Using the above test rig and after letting the v5.5 ESXi install settle for 5 minutes, the following can be seen:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi 5.5" src="/images/esxi-memory-house-load/esxiload05.png">

**ESXi 5.5 in house memory load is circa 1157MB**

### ESXi 6.0 Memory Load
Using the above test rig and after letting the v6.0 ESXi install settle for 5 minutes, the following can be seen:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi 6.0" src="/images/esxi-memory-house-load/esxiload06.png">

**ESXi 5.5 in house memory load is circa 1354MB**

## Results
Putting the above findings into a table, the following can be seen:
<div>
<style scoped>
table{
    margin: 0 auto;
    width: 70%;
    border-collapse: collapse;
    border-spacing: 0;
    border:1px solid #000000; }
th{
    text-align: center;
    padding: 2px;
    border:1px solid #000000; }
td{
    text-align: center;
    padding: 2px;
    border:1px solid #000000; }
tr:nth-child(even) {
    background-color: #efefef;}
</style>
</div>

| ESXi Version | Build Number | In House Load (MB) | Difference to v5.1 (MB) | Difference to v5.5 (MB) | Difference to v6.0  (MB) | 
|---- | ---- | ---- | ---- | ---- | ---- |
| 5.1 | 3070626 | 1052MB | 0 | -105 | -302 |
| 5.5 | 3343343 | 1157MB | +105 | 0 |-197 |
| 6.0 | 3380124 | 1354MB | +302 | +197 | 0 |

## Conclusions
So what conclusions can we draw from this testing?

Well,

- ESXi in house memory load does appear to increase in later versions
- The mean memory load increase across the 3 ESXi versions is in the region of 151MB per version when using the same hardware
- The in house memory load increase is pretty good going when you consider the new features that came with [vSphere 5.5](http://kb.vmware.com/kb/2058665) and [vSphere 6.0](http://kb.vmware.com/kb/2109816)  

Sure this testing was probably a bit sketchy and not all that indicative of ESXi running on actual hardware, but it is what it is and it is better than nothing at all.

As for my dilema of trying to upgrade yet not eat into too much of my current 512MB free memory, I think I'll upgrade my ESXi to v5.5 :o)

-Chris   

