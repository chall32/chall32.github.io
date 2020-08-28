---
layout: post
title: "vSphere VM Recovery on a Budget - Part 2" 
excerpt: "Site Pairing and Replication Configuration"
tags: 
- VMware
- Pro-Tip
image:
  thumb: budget-vm-recovery-pt1/budget-vm-recovery-00.png
comments: true
date: 2020-08-28T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="VM Recovery" src="/images/budget-vm-recovery-pt1/budget-vm-recovery-00.png">
Last time we deployed and configured our vSphere replication appliances, [catch up now](https://polarclouds.co.uk/budget-vm-recovery-pt1/). It's a great read. :wink:

This time we will configure our vSphere replication appliances and kick off some replications.

{% include _toc.html %}
## The Lab
We have vSphere replication deployed into my two site lab:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VRA Lab" src="/images/budget-vm-recovery-pt1/budget-vm-recovery-01.png">

Which looks like this in vCenter - Site Recovery:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Two Replication Appliances" src="/images/budget-vm-recovery-pt1/budget-vm-recovery-20.png">

## Site Pairing
Before we can replicate VMs across sites, we need to set up our site pairing. 

From vCenter - Site Recovery, click **OPEN Site Recovery** for our first site (lab: vc-site-a.lab, Site A):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Open SRA in Site A" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-01.png">

Click **New Site Pair**, select our local vCenter (lab: vc-site-a.lab), enter the details of our remote vCenter (lab: vc-site-b.lab) and click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Pair Sites 1" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-02.png">

If prompted, accept certificate warning.

Select second site vCenter along with it's replication appliance and click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Pair Sites 2" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-03.png">

Confirm configuration and click **Finish**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Pair Sites 3" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-04.png">

Back in Site Recovery, the site pair should now be detailed:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Pair Sites 4" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-05.png">

## Configure VM Replication
Now that we have the sites paired, let's get some VM replication up and running.

From Site Recovery, click **View Details** under the site pair created above. Confirm that the appliances are connected:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Sites Connected" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-06.png">

Click **Replications** and **New**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New VM Replication 1" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-07.png">

Confirm target site and click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New VM Replication 2" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-08.png">

Select a VM from the inventory and click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New VM Replication 3" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-09.png">

Select target datastore, modify disk format if required and click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New VM Replication 4" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-10.png">

Configure [RPO](https://en.wikipedia.org/wiki/Disaster_recovery#Recovery_Point_Objective), Enable point in time instances if required, enable network compression for VR data (recommended) as required and click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New VM Replication 5" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-11.png">

Review configuration and click **Finish**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New VM Replication 6" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-12.png">

Back under Replications - Outgoing, confirm that initial sync begins:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New VM Replication Initial Sync" src="/images/budget-vm-recovery-pt2/budget-vm-recovery2-13.png">

Repeat for any further VMs to be replicated.

## Conclusion and Wrap Up
In this post we paired our sites, configured vSphere replication and got our VM(s) replicated.

Next time we'll try to recover a replicated VM from a disaster... :dizzy_face: :astonished:

This was part 2 of a multipart series. If you missed part 1, [catch up now](https://polarclouds.co.uk/budget-vm-recovery-pt1/)!

-Chris









