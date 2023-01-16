---
layout: post
title: "NSX DFW System Excluded VM List Empty" 
excerpt: "Sawing Off the Branch You're Sitting On"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-00.png
comments: true
date: 2023-01-16T00:00:00+00:00
---
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Sawing Off the Branch" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-01.jpg">

{% include _toc.html %}
### Question: When is the worst time to find the NSX Distributed Firewall System Excluded VM list is empty?
Hint:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Setting Default Rules" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-02.png">

Yep that's it. When running NSX manager VM(s) on a cluster prepared for NSX and then setting the default NSX distributed firewall rules to **Drop** or **Reject**.

And that's when all hell doesn't break loose. Quite the opposite in fact.

## Oh F\/£k !!!!!1!!
<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX Manager Unknown" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-03.png">

Yeah... a whole lot of nothing going on.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="No ping!" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-04.png">

Yep. My NSX Manager is completely firewalled from the rest of my network and for all intents and purposes offline.

Choice words were spoken... Very choice words indeed...

## Background
Whilst writing an upcoming post on deploying NSX using the vCenter plug-in, I had my vSphere 8.0 and NSX 4.0.1.1 environment all in and spinning nicely. Final thing to talk about were the very bottom three rules of the application category - that is the very last three rules to be evaluated by the distributed firewall - which are set by default to allow any traffic from any source to any destination.

Yep, just set those to drop (or reject) and publish. 

Aaaand that is where we find ourselves; a firewalled NSX manager that we are unable to get at via the network to turn off said firewall.

## Recovery
So how do we recover from this situation?

How can we disable the NSX distributed firewall without using the NSX web GUI? 

From the NSX command line interface (CLI) perhaps?

What is the "magic" command to allow access again?

After much googling and CLI bashing, I discovered this command to be run on the ESXi server hosting my NSX Manager VM (this is a lab so I only have one manager - production environments should have three) to recover access to NSX Manager again:
{% highlight shell %}
vsipioctl clearallfilters -Override
{% endhighlight %}
Which when run on the ESXi host in question, looks like this (I suggest reading discussion points below BEFORE running):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="vsipioctl clearallfilters -Override" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-05.png">

I was then able to access my NSX Manager and change my default rules back.

For good measure after recovery I rebooted my ESXi host too.

## Recovery - Discussion Points
Some points to consider:
1. Migrate (vMotion) all VMs other than the affected NSX Manager VM(s) away from the ESXi host that the command is run on. This command will unilaterally remove **ALL firewall rules from ALL VMs** on the ESXi host it is run on.

2. Searching for the `vsipioctl clearallfilters` command in the VMware Knowledge Base results in just one hit: [vNICs are disconnected after NSX is uninstalled from ESXi (74556)](https://kb.vmware.com/s/article/74556){:target="_blank"}. At the time of writing this post, the related products section of this KB article lists only NSX for vSphere (aka NSX-v), not NSX-T or [NSX as it is now known as](https://blogs.vmware.com/partnernews/2022/04/nsx-data-center-name-change.html){:target="_blank"}.

3. After rebooting the ESXi host, you may find that firewall rules are still not being applied to VMs running on the . To resolve disconnect, save, reconnect and save each network connection for each VM affected:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Network Adapter" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-06.png">

Temporarily moving VMs to a different VDS port group and back may help too. Anything to get the ESXi host to rebuild it's dvfilter table.

## Making Sure This Does Not Happen Again
How can we make sure that this doesn't happen again? 

By making use of the user configured **Distributed Firewall Exclusion List**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Select Distributed Firewall Exclusion List" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-07.png">

Whilst we are here, a quick look in the **System Excluded VMs** list, shows nothing going on as suspected and experienced:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="System Excluded VMs List" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-08.png">

Back to **User Excluded Groups**, let's add, name and save our **DFW-Excluded** group:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Save Exclusion List" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-09.png">

Navigate to **Inventory > Groups** and edit the newly created **DFW-Excluded** group. Let's set some members:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Edit DFW-Excluded Group" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-10.png">

I'll add my NSX Manager by VM name:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add NSX Manager VM to DFW-Excluded" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-11.png">

As it's external to the NSX prepared environment (it's running on another ESXi host), I'll add my vCenter server by IP as well for good measure:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add vCenter IP to DFW-Excluded" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-12.png">

**Save** (twice) and lets look at the members of our DFW-Excluded group. First VMs:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="DFW-Excluded Member VMs" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-13.png">

Looks good. Checking IP addresses, we can see both the IPs of our vCenter and our NSX Manager listed (Remember production environments should have three NSX Managers deployed - this is just a lab): 

<img style="display: block; margin-left: auto; margin-right: auto;" alt="DFW-Excluded Member IPs" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-14.png">

## Making Sure This Does Not Happen Again - Testing
Let's pop in a distributed firewall rule to drop ping from any source to any destination:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ICMP any/any Drop" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-15.png">

Yep, we can still ping our NSX Manager:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ICMP Test" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-16.png">

Finally let's set the default rules to **Drop** again, and publish:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Set Default Rules to Drop Again" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-17.png">

A refresh of the browser to check that the vCenter NSX plug-in reloads...

<img style="display: block; margin-left: auto; margin-right: auto;" alt="All Good" src="/images/nsx-system-excluded-vm-list-empty/nsx-system-excluded-vm-list-empty-18.png">

Yep we are all good! Phew!

## Conclusion and Wrap Up
Well, a post that I did not think I would need to write. That said, a good post to have in the back pocket should I/you need to stop the NSX distributed firewall in a hurry in the future.

Certainly reading through the [Manage a Firewall Exclusion List](https://docs.vmware.com/en/VMware-NSX/4.0/administration/GUID-3B3C278D-4E35-4CE9-A4E2-ED6B1F25ABCE.html){:target="_blank"} section of the NSX documentation on the VMware website:
>NSX has system excluded virtual machines, and user excluded groups. NSX Manager and NSX Edge node virtual machines are automatically added to the read-only the System Excluded VMs list. 

Not in an NSX 4.0.1.1 vSphere 8.0 plug-in environment they are not! 

Finally as perhaps proof of a small world, take a look at the third command listed towards the bottom of this post: [NSX-T Nested ESXi Host Preparation Failed or Timed Out](/nsx-t-nested-host-prep-failed/){:target="_blank"}.

It would seem that I had the answer all along, yet I didn't know it. 

...story of my life... :wink:

-Chris