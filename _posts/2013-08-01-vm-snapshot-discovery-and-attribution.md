---
layout: post
title: VM Snapshot Discovery and Attribution
excerpt: "The Golden Snapshot Rule: VM SNAPSHOTS ARE NOT BACKUPS!"
tags:
- ESXi
- Reminder
- VMware
- ESX
image:
  thumb: vm-snapshot-discovery-and-attribution/snapshots.jpg
comments: true
date: 2013-08-01T12:43:00.000+01:00
---
The Golden Snapshot Rule:
<h1 style="text-align:center">VM SNAPSHOTS ARE NOT BACKUPS!</h1>

### What are VMware VM Snapshots?

Normal VM operation involves the virtual machine (VM) reading and writing to it's virtual disk (VMDK) file:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Normal" src="/images/vm-snapshot-discovery-and-attribution/Normal.jpg">

Upon the creation of a snapshot, the VM's virtual disk (VMDK) file is marked as read only. All changes are written to a snapshot log file, also known as a 'delta' file:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Snapshot" src="/images/vm-snapshot-discovery-and-attribution/Snapshot.jpg">

### So What is the Problem Here?

The problem is that these snapshot delta files left unchecked can grow and grow and grow, consuming more and more storage space.

### Surely VMware Have Some Guidelines Around VM Snapshots?

They do, and they are here: [http://kb.vmware.com/kb/1025279‎](http://kb.vmware.com/kb/1025279‎)

Lets pick up on some salient points here as it's worth repeating this as often as possible:

* Snapshots are not backups (Sound familiar?) 
* A snapshot file is only a change log of the original virtual disk
* Snapshots are not complete copies of the original vmdk disk files
* Use no single snapshot for more than 24-72 hours
* Regularly monitor systems configured for backups to ensure that no snapshots remain active for extensive periods of time
* An excessive number of delta files in a chain (caused by an excessive number of snapshots) or large delta files may cause decreased virtual machine and host performance
* If hosts and/or vCenter Server are prior to vSphere 5.0 confirm that there are no snapshots present (via command line) before a Storage vMotion
* Confirm that there are no snapshots present (via command line) before increasing the size of any virtual machine virtual disk or virtual RDM. If snapshots are present, delete them prior to increasing the size of the disk. Increasing the size of a disk with snapshots present can lead to corruption of snapshots and a potential data loss

### Got it. So How do I quickly and Simply Test for VM Snapshots?
Simple. This is where Chris' VM Snapshot Discovery and Attribution Tool comes in.
Here is a screenshot of the tool in action:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="SnapTool" src="/images/vm-snapshot-discovery-and-attribution/SnapTool.jpg">

So what do we have here?

Well, you can quite easily see that both the VM's SPONGEBOB and GARY have active snapshots. You can also see the details around these snapshots; their names, their descriptions and their sizes in GB.

What is super cool is we can also see who created them.  In the screenshot the snapshot creator is CHLABS\Chris (me!). OK, cool, but **think about it for a moment.**  If this was a production situation, it's more than possible that you will have multiple vSphere administrators.  Any one of these administrators can create snapshots.

Say for example I found that CHLABS\Fred.Bloggs was working on a some VMs, created several snapshots and had completed his changes.  Perhaps Fred did not know or understand The Golden Snapshot Rule.

With this newly discovered information now in hand, we can contact Fred, find out if he still needs those snapshots and perhaps educate him to the Golden Snapshot Rule.

Perhaps Fred forgot about the snapshots.......


#### Ah, the Forgotten Snapshot!
Don't joke.... it happens.
Where can I get a Copy of Chris' VM Snapshot Discovery and Attribution Tool?

Simple. Grab your copy here: [https://github.com/chall32/GetSnapshot](https://github.com/chall32/GetSnapshot)

### So I Have VMs With Snapshots. What To Do?

Here are your options:
<div>
<style scoped>
table{
    width: 70%;
    border-collapse: collapse;
    border-spacing: 0;
    border:1px solid #000000; }
th{
    text-align: center;
    border:1px solid #000000; }
td{
    text-align: left;
    border:1px solid #000000;}
tr:nth-child(even) {
    background-color: #efefef;}
</style>
</div>

| Snapshot Operation | Effect
|---------|--------|
|Take | The current state of the virtual machine and its guest operating system is captured.
|Revert | The state of the virtual machine and its guest operating system reverts back to what it was when a snapshot was taken. If there are multiple snapshots, the snapshot taken immediately prior to the current state is used. **Warning: All current data is permanently lost**.
|Delete | The state of the virtual machine is changed to the current state (that is, changes made after taking the snapshot are saved to the base disk). In earlier versions of some products the menu option is named **Remove**.
|Delete (Snapshot Manager) | The state of the virtual machine is changed to the current state (that is, changes made after taking the snapshot are saved to the base disk). The snapshot chosen to be deleted is available for selection in a graphical display that shows all existing snapshots. This is available only in products that support multiple snapshots.
|Go To (Snapshot Manager) | The state of the virtual machine and its current guest operating system switches to the state of that of an arbitrarily chosen snapshot. The snapshot chosen to switch to is available for selection in a graphical display that shows all existing snapshots. This is available only in products that support multiple snapshots.

May I recommend the Delete option?
Sure it doesn't feel right to click "Delete" to carry on as normal with the VM, but it is the correct option!

### What Can I Do Longer Term to Prevent Forgotten Snapshots?
Have a look at [http://kb.vmware.com/kb/1018029](http://kb.vmware.com/kb/1018029)
This VMware KB article shows you how to configure VMware vCenter Server to send alerts when virtual machines are running from snapshots.
Conclusion & Troubleshooting

You now know all about VM snapshots, how to test for them, how to find out who created them, and how to delete them.

If you need to troubleshoot any issues with VM snapshots, have a look at the bottom of [http://kb.vmware.com/kb/1025279‎](http://kb.vmware.com/kb/1025279). There are plenty of resources to look at.

-Chris
