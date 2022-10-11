---
layout: post
title: "vSphere and ESXi 8.0: The Unsupported" 
excerpt: "Gone But Not Forgotten?"
tags: 
- Pro-Tip
- VMware
- ESXi
image:
  thumb: /esxi-8-the-unsupported/esxi-8-the-unsupported-01.png
comments: true
date: 2022-10-11T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/esxi-8-the-unsupported/esxi-8-the-unsupported-01.png">
With the release of any software major version update, there are always those that get left behind.

Familiar readers will remember my own struggles with vSphere 7.0: [Workaround ESXi CPU Unsupported Error](/workaround-esxi-cpu-unsupported/){:target="_blank"}, [ESXi 7.0: The Missing PERC(s)](/esxi7-missing-percs/){:target="_blank"} and finally [ESXi 7.0: The Unsupported](/esxi-7-the-unsupported/){:target="_blank"}. Fun times!

This time around with the release of vSphere 8.0 (containing ESXi 8.0), lets take another look at the gone but not forgotten hardware no longer supported by VMware's latest vSphere release. 

As usual, the [VMware vSphere 8.0 Release Notes](https://docs.vmware.com/en/VMware-vSphere/8.0/rn/vmware-vsphere-80-release-notes/index.html){:target="_blank"} are a good place to start.

## VMware Knowledge Base
{% include _toc.html %}
Rather than having to poke around in a Dell OEM ISO image as I did for ESXi 7.0, this time around VMware have been quite open an honest about deprecated and unsupported devices in ESXi 8.0. Further information can be found in  [Devices deprecated and unsupported in ESXi 8.0 (88172)](https://kb.vmware.com/s/article/88172){:target="_blank"}.

Attached to the above KB (right hand side, second box down), you will find a tabbed spreadsheet that contains the details of devices no longer supported in ESXi 8.0.

That said, one thing missing from the spreadsheet is unsupported CPUs. 

Don't worry, the VMware KB has your back for that too: [Updated Plan for CPU Support Discontinuation In Future Major vSphere Releases (82794)](https://kb.vmware.com/s/article/82794){:target="_blank"}.

## VMware Compatibility Guide
For the latest and greatest advice, consult the [VMware Compatibility Guide](https://www.vmware.com/resources/compatibility/search.php){:target="_blank"} for details. Simply search for your device(s) to confirm compatibility.

Not much more to be said other than this should be your number one source for the latest compatibility and support news.

## VMware Interoperability Matrix
If an existing vSphere deployment leverages other VMware products such as (for example) NSX, vCloud Director, Site Recovery Manager, (to name just a few), often the compatibility and interoperability of these products needs to be taken into account prior to upgrading to vSphere 8.0.

To avoid issues, consult the [VMware Product Interoperability Matrix](https://interopmatrix.vmware.com){:target="_blank"} for details. Not only does the matrix provide interoperability details, it can also provide upgrade paths to get you where you want to be.

## Try It
Given that ESXi supports being installed to a USB device, use a spare machine to install ESXi onto a USB stick (minimum 32GB capacity stick recommended), transfer the USB stick to the server to be tested and reboot from USB.

VMware Workstation can also be used to create the ESXi 8.0 USB stick. Simply connect the USB to a VM and install ESXi to it:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Workstation: Connect USB" src="/images/esxi-8-the-unsupported/esxi-8-the-unsupported-02.png">

## Workarounds 
**NOTE:** [Usual disclaimer applies](/pages/disclaimer/){:target="_blank"}. Needless to say that some/all of these workarounds may not be supported by VMware either.

### Bypass Unsupported CPU Check
The following kernel option can be added to bypass the unsupported CPU check during ESXi installation or upgrade:
{% highlight shell %}
allowLegacyCPU=true
{% endhighlight %}
See [this post](https://williamlam.com/2020/04/quick-tip-allow-unsupported-cpus-when-upgrading-to-esxi-7-0.html){:target="_blank"} on William Lam's blog for further details.

### Apple and Lenovo Hardware
The following kernel option can be added to allow ESXi 8.0 to boot on some Apple hardware:
{% highlight shell %}
norts=1
{% endhighlight %}
For Apple hardware, see [this post](https://williamlam.com/2022/10/vsphere-8-on-apple-mac-hardware.html){:target="_blank"} on William Lam's blog for further details.<br>
For Lenovo hardware, see [this how to](https://support.lenovo.com/gb/en//solutions/ht510589-system-stuck-at-shutting-down-firmware-services-when-installing-vmware-esxi-65-u3-through-uefi-mode-ipxe-lenovo-thinksystem/){:target="_blank"} for further details.

## Replace Unsupported Hardware
Unfortunately "rip and replace" may be the only option, as discussed in my post [ESXi 7.0: The Missing PERC(s) - Part 2](/esxi7-missing-percs-pt2/){:target="_blank"}. 

However, given ongoing supply chain issues, simply "dropping in" a brand new vSphere farm to run vSphere 8.0 may prove to not be such a quick fix as it may have once been.

For production systems wanting to keep VMware support, then 100% supported hardware is a must, even if that means running a down-level vSphere version until end of system life.

For home and lab users a little more flexibility can be afforded. Sure a particular server model may not be listed as supported in the VMware Compatibility Guide, however if it's component parts (CPUs, storage controllers, network cards, etc) are listed / supported and found to work, then hey go for it and install ESXi 8.0.

Remember: ebay is your friend! *(Other sources of server hardware are available)*.

**Pro-Tip**: Checkout [LabGopher](https://labgopher.com/){:target="_blank"} for second hand server deals. See the links on LabGopher for localised (US, UK, Canada and Australia) sites. 

## Migrate to the Cloud
Whilst not a maybe not solution for everyone, functionality exists for existing VMware VMs to be run in the cloud with minimal reconfiguration:

- vSphere on [Amazon AWS](https://aws.amazon.com/vmware/){:target="_blank"}
- vSphere on [Microsoft Azure](https://azure.microsoft.com/en-gb/products/azure-vmware/){:target="_blank"}
- vSphere on [Google Cloud](https://cloud.google.com/vmware-engine){:target="_blank"}

Once VMs are running it the cloud, unsupported on-premises hardware can be decommissioned.

## Other Hypervisors
Shocker I know, other server based Hypervisors do exist! They often do not have the same limitations to them that vSphere 8.0 does. These roughly drop into two categories. Here are three of each.

### Commercially Supported
- [Microsoft Azure Stack HCI](https://learn.microsoft.com/en-us/azure-stack/hci/overview){:target="_blank"} - Microsoft's on-premises infrastructure with Azure cloud services
- [Nutanix](https://www.nutanix.com){:target="_blank"} - Hyper-converged on-premises infrastructure
- [Oracle Virtualisation](https://www.oracle.com/virtualization/){:target="_blank"} - On-premises virtualisation based on Linux KVM

What about Microsoft Hyper-V? As discussed [here](https://www.theregister.com/2021/08/31/hyper_v_server_discontinued/){:target="_blank"} on the Register:
> Azure Stack HCI is Microsoft's premier hypervisor offering for running virtual machines on-premises

Hyper-V's days are numbered it would seem.

### Free Open Source with Support Available
- [Proxmox](https://www.proxmox.com){:target="_blank"} - All in one solution, popular with home lab-ers
- [XCP-ng](https://xcp-ng.org/){:target="_blank"} - Provides separate host (XCP-ng) and host management (Xen Orchestra) experience 
- [Linux KVM](https://www.linux-kvm.org/page/Main_Page){:target="_blank"} - The basis of Proxmox, XCP-ng and Oracle Virtualisation. For the purists!

## Conclusion and Wrap Up
In this post we looked at avoiding the pitfalls that can be fallen into when upgrading to vSphere 8.0.  Following on from there we looked at ways of working around and upgrading hardware to allow ESXi 8.0 to be installed. Via a quick trip to running vSphere VMs natively on the cloud we looked at alternative type-1 hypervisor solutions; both commercial and free.  

Comparing today (October 2022) to where we were when vSphere 7.0 was launched (April 2020), it feels that whilst there are squeezes on traditional on-premises vSphere deployments from both cloud and supply chain issues, there are more options open to clients now then there were at the time of the vSphere 7.0 launch. 

Also as I briefly touched on in the introduction of my [VMware Skyline Health Diagnostics Tool](/skyline-health-diag-tool/){:target="_blank"} post, there are clients that find themselves not able to move to cloud whilst simultaneously unable upgrade existing hardware. They are locked into their current (supported or unsupported) vSphere version for the time being. 

Finally there is the [Broadcom acquisition of VMware](https://www.broadcom.com/company/news/financial-releases/60271){:target="_blank"} and the uncertainty that it brings. 

The overall uptake of vSphere 8.0 is going to be interesting. 

-Chris