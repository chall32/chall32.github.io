---
layout: post
title: "Failed to Expand VMFS Datastore - Cannot Change the Host Configuration" 
excerpt: "Problems Expanding ESXi VMFS Partitions"
tags: 
- VMware
- Pro-Tip
image:
  thumb: failed-to-expand-vmfs-datastore/failed-to-expand-01.png
comments: true
date: 2020-09-16T01:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-01.png">
Spun up a quick nested ESXi 7.0 VM for some testing and noticed I needed to expand VMFS datatstore to fit my VMs. As is usual, I'd sized my VM on the slightly small side.  

*Rather than 'being tight' with vSphere resources, I like to think of it as **being frugal** when allocating resources to VMs. The more frugal you are with resources when allocating to VMs, the more VMs you can get into the environment.<br>
More VMs equals more fun!* :smiley:

Anyway, back to the topic in hand.

Simple enough I thought, I'll increase the nested ESXi's diskspace and then expand the VMFS datastore into the newly added space. One, two, three, and oh... 

Here is a screenshot of the error I encountered:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Failed to Expand Error Close Up" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-00.png">

> Failed to expand VMFS datastore datatstore1 - Cannot change the host configuration.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Failed to Expand Error" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-02.png">

So how did we get to this error? Is there a way to expand the VMFS datastore?<br>

**Spoiler Alert:** Yes! There is a way to expand the VMFS, and no you don't need to trash your datastore first!
{% include _toc.html %}
## Reproducing the Error
Simple.  Try to expand a VMFS datastore that was created on the boot disks of an ESXi install using the host web client:

Step 1 - select **Increase capacity**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Step 1" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-03.png">

Step 2 - select **Expand an existing VMFS darastore**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Step 2" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-04.png">

Step 3 - select the datastore to expand:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Step 3" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-05.png">

Step 4 - move the slider to allocate the free space to the VMFS partition:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Step 4" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-06.png">

Step 5 - confirm and click **Finish**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Step 5" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-07.png">

Oh look an error:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Step 6 - Error" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-08.png">

Doh!

## [Solved] How to Expand the Datastore
So if we can't expand a boot drive datastore through the host web client, we are going to have to drop to the command line to achieve the expansion.

:warning: ***Standard disclaimers apply!  Proceed at your own risk. Follows is for information only.  YOU CONTROL YOUR DATA! Back it up first perhaps?*** :warning:	

First, we need some information regarding the disks upon which the partitions are located and the partition number of the VMFS datastore to be expanded. Simple enough; browse to the datastore in the web client and find it's extent details:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VMFS Info" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-15.png">

<code>Extent 0: mpx.vmhba0:C0:T0:L0, partition 8</code> is the information we are interested in.

Next, lets fire up an SSH session or use the [DCUI](https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.vsphere.security.doc/GUID-94F0C54F-05E3-4E16-8027-0280B9ED1009.html) doesn't matter which.

Lets use the information gained above to get some further info on the partition table:

{% highlight text %}
partedUtil getptbl "/vmfs/devices/disks/DeviceName"
{% endhighlight %}

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Get Partition Table" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-09.png">

Yep, <code>8 268437504 419430366 AA31E02A400F11DB9590000C2911D1B8 vmfs 0</code> is the VMFS partition we want to work on.

Next, lets confirm that the partition structure is OK before we make any changes:

{% highlight text %}
partedUtil fixGpt "/vmfs/devices/disks/DeviceName"
{% endhighlight %}

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Fix Partition Table" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-10.png">

Perfect.  Next, let's find the starting sector of our VMFS partition.

As we can see from the above screenshot, the first number after the partition number of our VMFS in the above figure is our starting sector:<br>
<code>8 <mark>268437504</mark> 419430366 AA31E02A400F11DB9590000C2911D1B8 vmfs 0</code>

Our VMFS partition starts at sector 268437504.

Next, let's find the end usable sector on the disk:

{% highlight text %}
partedUtil getUsableSectors "/vmfs/devices/disks/DeviceName"
{% endhighlight %}

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Get Usable Sectors" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-11.png">

As we can see from the above screenshot, the second number returned is the last usable sector on the disk:
<code>34 <mark>524287966</mark></code>

524287966 is the last usable sector on our disk.

So putting the info gleamed from the previous commands, we easily can construct the partition expand command. Here is the syntax:

{% highlight text %}
partedUtil resize "/vmfs/devices/disks/DeviceName" PartitionNumber NewStartSector NewEndSector
{% endhighlight %}
Using the above information, our command will look like this:
{% highlight text %}
partedUtil resize "/vmfs/devices/disks/mpx.vmhba0:C0:T0:L0" 8 268437504 524287966
{% endhighlight %}
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Resize" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-12.png">

Nice. Partition expanded. :thumbsup:

Next we need to expand the VMFS file system into the expanded partition.  Here is the syntax:

{% highlight text %}
vmkfstools --growfs "/vmfs/devices/disks/DeviceName:PartitionNumber" "/vmfs/devices/disks/DeviceName:PartitionNumber"
{% endhighlight %}
Using the above information, our command will look like this:
{% highlight text %}
vmkfstools --growfs "/vmfs/devices/disks/mpx.vmhba0:C0:T0:L0:8" "/vmfs/devices/disks/mpx.vmhba0:C0:T0:L0:8"
{% endhighlight %}
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Grow VMFS" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-13.png">

After refreshing the web client:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Job Done" src="/images/failed-to-expand-vmfs-datastore/failed-to-expand-14.png">

BOOM! :boom: Job done! :thumbsup: :sunglasses: :thumbsup: 

All in all, not difficult if you know which commands to run and where to glean the information to put into the commands. 

For more information on the ESXi partedUtil command line utility, take a look at [VMware KB 1036609](https://kb.vmware.com/s/article/1036609).

-Chris