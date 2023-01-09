---
layout: post
title: "VMs with TPMs" 
excerpt: "Properly Provisioned"
tags: 
- Pro-Tip
- VMware
- ESXi
image:
  thumb: /vms-with-tpms/vms-with-tpms-01.png
comments: true
date: 2023-01-09T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="ESXi + Key" src="/images/vms-with-tpms/vms-with-tpms-01.png">
A quick post to show how to handle the creation of Virtual Machines (VMs) that require Trusted Platform Modules (TPMs) to function.

A Trusted Platform Module, or TPM, is a secure crypto processor that secures a computer via an integrated cryptographic key. But in more basic terms, it's like a security alarm for your computer (or virtual machine) to prevent hackers or malware from accessing data.

{% include _toc.html %} 
## TPMs and vTPMs
A TPM is a requirement run some modern operating systems such as Windows 11 ([Windows 11 requirements](https://www.microsoft.com/en-gb/windows/windows-11-specifications){:target="_blank"}) without workarounds.  Therefore to be able to run, for example Windows 11 as a virtual machine, our VM is going to need a Virtual TPM or vTPM.

Let's look closer at creating a VM with a VTPM.

## Create vSphere Native Key Provider
Before we can provision VMs with vTPMs, we need a key provider. For deployments of vSphere 7.0 update 2 or later, vCenter has a key provider built in. VMware calls this the vSphere Native Key Provider. 

For an overview of the vSphere Native Key Provider, see the Native Key Provider [documentation](https://docs.vmware.com/en/VMware-vSphere/8.0/vsphere-security/GUID-54B9FBA2-FDB1-400B-A6AE-81BF3AC9DF97.html){:target="_blank"}.

For the purposes of this post, let's set up a Native Key Provider. 

From the vSphere client select the vCenter instance at the top of the inventory list. Then select **Configure > Security > Add > Add Native Key Provider**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Native Key Provider 1" src="/images/vms-with-tpms/vms-with-tpms-02.png">

Name the provider:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Native Key Provider 2" src="/images/vms-with-tpms/vms-with-tpms-03.png">

After creation of the provider, but before we can use it, we need to back up the provider configuration. Select **Back Up** to continue:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Back Up Provider 1" src="/images/vms-with-tpms/vms-with-tpms-04.png">

Supply a suitably complex password and select **Back Up Key Provider**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Back Up Provider 2" src="/images/vms-with-tpms/vms-with-tpms-05.png">

Once the backup completes, the provider becomes active and available for use:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Native Provider Provisioned" src="/images/vms-with-tpms/vms-with-tpms-06.png">

## Provision a VM with a vTPM
Right. Let's create a Windows 11 VM. For brevity, I'll cover just the salient vTPM points below.

During the Windows 11 VM creation, we can see that the VM will be provided with a vTPM:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add TPM" src="/images/vms-with-tpms/vms-with-tpms-07.png">

Confirming the VM configuration prior to completion, all looks good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="TPM Present" src="/images/vms-with-tpms/vms-with-tpms-08.png">

Let's fire our VM up:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="TPM Enabled Boot" src="/images/vms-with-tpms/vms-with-tpms-09.png">

Install Windows 11:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Win11 Installing" src="/images/vms-with-tpms/vms-with-tpms-10.png">

Looks good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Win11 Installed" src="/images/vms-with-tpms/vms-with-tpms-11.png">

Checking our VM details, we can see that the VM is encrypted with a native key provider:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Encrypted with Native Key Provider" src="/images/vms-with-tpms/vms-with-tpms-12.png">

Looking at the virtual hardware from the Windows install within the VM, a TPM can be seen:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Win11 Devices - TPM 2.0" src="/images/vms-with-tpms/vms-with-tpms-13.png">


And that's all there is to it!  Simple, easy peasy!

## Deletion of a Key Provider 
In the interests of "I wonder what happens when...", let's simulate the loss of a Key Provider. First, I'll delete my Key Provider:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Delete Key Provider" src="/images/vms-with-tpms/vms-with-tpms-14.png">

Let's see if our Windows VM will continue to operate correctly. Powering on:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Encrypted VM Power On without NKP 1" src="/images/vms-with-tpms/vms-with-tpms-15.png">

Yep, VM is starting up OK:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Encrypted VM Power On without NKP 2" src="/images/vms-with-tpms/vms-with-tpms-16.png">

OK, I'll power off and remove it from the vCenter inventory:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remove Encrypted VM" src="/images/vms-with-tpms/vms-with-tpms-17.png">

Next, I'll re-register the VM back into the vCenter inventory:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Register Encrypted VM" src="/images/vms-with-tpms/vms-with-tpms-18.png">

From the start the VM is marked as invalid:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Invalid Encrypted VM" src="/images/vms-with-tpms/vms-with-tpms-19.png">

After sometime, I receive a "Virtual Machine Locked Alarm". OK, let's try an **Unlock VM**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Invalid Encrypted VM Unlock - No NKP 1" src="/images/vms-with-tpms/vms-with-tpms-20.png">

OK, vCenter will try to transmit an encryption key to my VM:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Invalid Encrypted VM Unlock - No NKP 2" src="/images/vms-with-tpms/vms-with-tpms-21.png">

As suspected, because the Site-A-Key-Provider no longer exists, I cannot unlock the VM in order to power it on:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Invalid Encrypted VM Unlock - No NKP 3" src="/images/vms-with-tpms/vms-with-tpms-22.png">

## Restore Key Provider
Remember the backup we took before we were able to complete the creation of the Native Key Provider? 

Let's restore that backup now:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Restore Native Key Provider 1" src="/images/vms-with-tpms/vms-with-tpms-23.png">

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Restore Native Key Provider 2" src="/images/vms-with-tpms/vms-with-tpms-24.png">

With the key provider restored, let's try to unlock our VM again:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Invalid Encrypted VM Unlock - With NKP 1" src="/images/vms-with-tpms/vms-with-tpms-25.png">

Finally, let's power the VM on:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Invalid Encrypted VM Unlock - With NKP 2" src="/images/vms-with-tpms/vms-with-tpms-26.png">

Looks good. Our Windows 11 VM is able to boot again.

## Conclusion and Wrap Up
With a bit of up front configuration, deploying TPM enabled modern operating system based VMs such as Windows 11 are simple enough to complete when using the vSphere Native Key Provider. 

-Chris