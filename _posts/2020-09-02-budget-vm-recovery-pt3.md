---
layout: post
title: "vSphere VM Recovery on a Budget - Part 3" 
excerpt: "Disaster Strikes!"
tags: 
- VMware
- Pro-Tip
image:
  thumb: budget-vm-recovery-pt1/budget-vm-recovery-00.png
comments: true
date: 2020-09-02T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="VM Recovery" src="/images/budget-vm-recovery-pt1/budget-vm-recovery-00.png">
Last time we paired our sites and kicked off some VM replication, [catch up now](https://polarclouds.co.uk/budget-vm-recovery-pt2/). It's a great read. :wink:

This time we will look at how to recover from a disaster using our replicated VM.

{% include _toc.html %}
<br>First off, let's double check that our VM is still being replicated.  Yep looks good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 1" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-01.png">

## Oooops, I broke It!
Let's ~~internationally~~ accidently cause a disaster by breaking our replicated VM.  Shall we uninstall the storage controller driver? Yes, let's:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 2" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-02.png">

For good measure, let's rename the driver file too. *Belt and braces breakage* :wink:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 3" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-03.png">

Cheeky reboot... **Oh noes! :dizzy_face: It's BSODing broke:**

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 4" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-04.png">

Oh dear! Looks like I need to recover my VM! :astonished: 

## Step 1: Recovery to Secondary Site
Firstly **power off the failed VM**.  Don't delete the failed VM from the inventory. We'll need it later.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Dead Already" src="/images/budget-vm-recovery-pt3/DeadAlready.png">
*Leave it... Besides, <a href="https://www.youtube.com/watch?v=fHAOWLhrxhQ" target="_blank">it's dead already</a>*
 
Next, let's head into Site Recovery on Site B, navigate to incoming replications and select **Recover**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 5" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-05.png">

As our source VM is dead (Jim), let's recover from the latest data already replicated to Site B: 

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 6" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-06.png">

We'll select Site B to house our recovered VM:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 7" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-07.png">

Select our Site B ESXi Host:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 8" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-08.png">

And finish. For the moment, we don't care that our recovered VM will be disconnected from the network:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 9" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-09.png">

Boom! Recovery to Site B complete:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 10" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-10.png">

Power on the recovered VM in Site B and let's see if it boots OK. Looks good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 11" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-11.png">

If it didn't boot for whatever reason, we have the opportunity to go back in time via reverting to snapshots taken as per our replication interval setup in [Part 2](https://polarclouds.co.uk/budget-vm-recovery-pt2/#configure-vm-replication):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 12" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-12.png">

As we are good, let's delete those old snapshots:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 13" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-13.png">

As we have recovered the VM into site B and all is good, let's clean up replication:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 14" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-14.png">

Gracefully:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 15" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-15.png">

## Step 2: Replicate Changes Back to Primary Site
So we have a good VM, but it's running off the network in our secondary site.  How do we get it back onto our primary site and back on the network?

Here's how.

Firstly, login to the site recovery on the secondary site. From there, select **Replications - Outgoing - New**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 16" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-16.png">

Setup replication from secondary site back to primary site, as previously completed in [Part 2](https://polarclouds.co.uk/budget-vm-recovery-pt2/#configure-vm-replication), however when prompted tick **Select seeds**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 17" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-17.png">

Selecting seeds will allow us to compare our working VM on our secondary site with the previously failed VM on primary site and replicate **only the changes** back to the primary site. Replicating just the changes back to our primary site will save on both time and network bandwidth.

Confirm seeds are correct and tick the confirmation box:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 18" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-18.png">

Configure replication settings:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 19" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-19.png">

Review and click **Finish**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 20" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-20.png">

As can be seen, only changes are replicated back to the primary site:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 21" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-21.png">

Replication from secondary site to primary site complete:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 22" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-22.png">

## Step 3: Power on Recovered VM 
Let's power on our recovered VM back on our primary site. Looks good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 23" 
src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-23.png">

Checking for snapshots, there are none:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 24" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-24.png">

VM recovered, back on the network, service restored, day saved, everyone happy, :sunglasses: bonus payment in the post :moneybag::moneybag:

## Post Recovery Clean Up and Reprotection
A couple of house keeping jobs now that our VM bas been recovered.

### Clean Up Secondary Site 
Let's power off the replica on our secondary site:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 25" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-25.png">

To avoid future confusion, let's remove it from the secondary site inventory.  

As you'll see below, just removing the VM from the inventory rather than deleting it will save us time and network bandwidth later on:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 26" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-26.png">

Yep, confirm removal from inventory:

 
 <img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 27" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-27.png">

### Reprotection of Primary VM
As we've already covered configuring replication in [Part 2](https://polarclouds.co.uk/budget-vm-recovery-pt2/#configure-vm-replication), using seeds [just above](https://polarclouds.co.uk/budget-vm-recovery-pt3/#step-2-replicate-changes-back-to-primary-site), I won't cover that again.  Suffice to say that replicating just the initial changes back to the secondary site will save both time and bandwidth (but you knew that already :wink:):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 28" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-28.png">

aaaand we're done:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Part 3 - 29" src="/images/budget-vm-recovery-pt3/budget-vm-recovery3-29.png">

Not only has our primary site production VM recovered, it's also being replicated to our secondary site just as it was at the beginning of this post.

## Conclusion and Wrap Up
So there we have it. In this series we (click a link to take another look):

- [Discovered vSphere Replication](https://polarclouds.co.uk/budget-vm-recovery-pt1/)
- [Setup vSphere Replication appliances](https://polarclouds.co.uk/budget-vm-recovery-pt1/#deploying-vsphere-recovery-appliances)
- [Configured VM replication](https://polarclouds.co.uk/budget-vm-recovery-pt2/#configure-vm-replication)
- [Killed our primary VM](https://polarclouds.co.uk/budget-vm-recovery-pt3/#oooops-i-broke-it)
- [Recovered our failed VM to our secondary / recovery site](https://polarclouds.co.uk/budget-vm-recovery-pt3/#step-1-recovery-to-secondary-site)
- [Replicated our recovered VM back to our primary site](https://polarclouds.co.uk/budget-vm-recovery-pt3/#step-2-replicate-changes-back-to-primary-site)
- [Reprotected our primary VM again](https://polarclouds.co.uk/budget-vm-recovery-pt3/#post-recovery-clean-up-and-reprotection)

Links to the other parts of series are as follows:

- Part 1: [No money, no problem: Introduction and Deployment](https://polarclouds.co.uk/budget-vm-recovery-pt1/)
- Part 2: [Site Pairing and Replication Configuration](https://polarclouds.co.uk/budget-vm-recovery-pt2/)
- Part 3: This part - Disaster Strikes!

As we saw, recovering replicated VMs from disaster is quite a simple process.

Until next time :thumbsup::sunglasses::thumbsup:

-Chris
