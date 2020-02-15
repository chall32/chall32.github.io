---
layout: post
title: "VMware NSX for vSphere Upgrade Notes" 
excerpt: "Upgrading NSX in a cross-vCenter environment. Easy, unless..."
tags: 
- VMware
- Pro-Tip
image:
  thumb: nsx-upgrade/nsx-upgrade-00.jpg
comments: true
date: 2020-02-15T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="pfSense + OpenVPN" src="/images/nsx-upgrade/nsx-upgrade-00.jpg">
Some quick notes and a nice work around for achieving a smooth NSX for vSphere upgrade as gleaned from upgrading my cross-vCenter home lab from v6.4.5 to v6.4.6.

{% include _toc.html %}
## Pre-Requisites
Check the VMware Product Interoperability Matrices.  Check [this matrix](https://www.vmware.com/resources/compatibility/sim/interop_matrix.php#interop&93=&2=&1=) for NSX/vCenter/ESXi interoperability.

Ensure that your vCenter and ESXi servers are running compatible versions prior to starting the NSX upgrade!

Check the upgrade release notes. See [here](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/rn/releasenotes_nsx_vsphere_646.html) for all VMware NSX Data Center for vSphere release notes

## Download
Download the upgrade bundle from [the VMware download site](https://my.vmware.com/web/vmware/details?downloadGroup=NSXV_646&productId=491)

Once downloaded, confirm the check sum using PowerShell:
```powershell
get-filehash VMware-NSX-Manager-upgrade-bundle-6.4.6-14819921.tar.gz
```
Confirm SHA256SUM value matches that given on the VMware download site

## Upgrade Steps
NSX Data Center for vSphere components **must** be upgraded in the following order:
1. Primary NSX Manager appliance
2. All secondary NSX Manager appliances
3. NSX Controller cluster
4. Host clusters
5. NSX Edge
6. Guest Introspection
7. Post-Upgrade Tasks

The top level VMware upgrade guide is [available here](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-D824C743-8137-47F8-AF5D-C225CC8A2542.html)

### Upgrade Primary NSX Appliance
**When:** Anytime <br> 
**How:** Follow [this guide](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-D05908F8-AB87-474B-9522-BACDD685D827.html) <br>
**Further Notes:** None

### Upgrade All Secondary NSX Manager Appliances
**When:** Anytime <br> 
**How:** Follow [this guide](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-C3DE6069-540E-4EF5-84AC-5EC12752EC6C.html) <br>
**Further Notes:** None

### Upgrade NSX Controller Cluster
**When:** During a maintenance window <br>
**How:** Follow [this guide](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-50663A38-C1C5-4D07-A203-9000F4D9FBFA.html) <br>
**Further Notes:** The VMware guidance of performing the upgrade "during a maintenance window" seems a little conservative.  As long as there are no NSX changes being made whilst the upgrade is taking place, then I suggest that this can be done anytime

### Upgrade Host Clusters
**When:** During a maintenance window <br>
**How:** Follow [this guide](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-8355030A-9FF0-4C44-B694-42F655B76BA0.html) <br>
**Further Notes:** This is where my one host per cluster lab had issues.<br>
The problem was that the upgrade process wanted to put the host into maintenance mode to upgrade the ESXi NSX VIBs but couldn't - because it had no where to vMotion the VMs to! (remember - only one host per cluster!) :flushed:

#### Work Around
**_What follows is the NSX VIB upgrade process if for single host "lab" style clusters. Certainly in production you will have more than one ESXi host per NSX cluster, so this work around would (should) not be needed._**

The solution was to upgrade the NSX VIBs manually, and then reboot the host and all it's VMs to enable the updated NSX VIBs.  Luckily I found [this post](https://www.definetomorrow.co.uk/blog/2018/5/10/manual-install-of-nsx-vibs-to-esxi-hosts) that details a method to obtain the updated NSX VIBs from NSX manager. 

For my lab upgrade to 6.4.6, I Opened  https://nsx-site-a.lab/bin/bin/vdn/nwfabric.properties in a web browser to find the updated NSX VIBs:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VIB Locations" src="/images/nsx-upgrade/nsx-upgrade-01.png">

Downloaded the VIB from https://nsx-site-a.lab/bin/vdn/vibs-6.4.6/6.7-14762108/vxlan.zip using IE as Chrome would just open the file

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Download VIB" src="/images/nsx-upgrade/nsx-upgrade-02.png">

Extracted the VIB from the folder \vib20\esx-nsxv inside the zip:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Extract VIB" src="/images/nsx-upgrade/nsx-upgrade-03.png">

WinSCP'ed the VMware_bootbank_esx-nsxv_6.7.0-0.0.14762108.vib file to the ESXi host (after enabling SSH on the ESXi host)

<img style="display: block; margin-left: auto; margin-right: auto;" alt="WinSCP VIB" src="/images/nsx-upgrade/nsx-upgrade-04.png">

Shutdown all VMs, placed host into maintenance mode, SSH'ed to the host and upgraded the NSX VIB using the following command:
```bash
# esxcli software vib update -v /tmp/VMware_bootbank_esx-nsxv_6.7.0-0.0.14762108.vib
```
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Upgrade VIB" src="/images/nsx-upgrade/nsx-upgrade-05.png">

Rebooted my ESXi host, removed host from maintenance mode, booted all the VMs and boom! :boom:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VIB Done" src="/images/nsx-upgrade/nsx-upgrade-06.png">

Repeat for the other ESXi host at the other site. Double boom! :boom: :boom:

### Upgrade NSX Edges
**When:** During a maintenance window <br> 
**How:** Follow [this guide](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-17457D8A-471B-4FF9-9372-7079F6AD47C4.html) <br>
**Further Notes:** More of an edge replacement process; a new Edge virtual appliance is deployed alongside the existing one. When the new Edge is ready, the old Edge's vNICs are disconnected and the new Edge's vNICs are connected.

### Upgrade Guest Introspection
**When:** During a maintenance window <br> 
**How:** Follow [this guide](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-786B7082-BCF1-4130-97C1-8128DF58F567.html) <br>
**Further Notes:** I don't run guest introspection in my lab, so did not complete this step

### Upgrade NSX Services That Do Not Support Direct Upgrade
**When:** Depends on service(s)<br> 
**How:** Follow [this guide](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-F0D75A47-8AAD-47FC-B69D-72775212A65A_copy2.html) <br>
**Further Notes:** Nothing to do here as I don't run any other NSX related services

### Post-Upgrade Tasks
**When:** Anytime <br> 
**How:** Follow [this guide](https://docs.vmware.com/en/VMware-NSX-Data-Center-for-vSphere/6.4/com.vmware.nsx.upgrade.doc/GUID-27BED4A8-9E87-4D42-AEEC-ABB09B6F1762_copy2.html) <br>
**Further Notes:** We know that our VIBs are OK as we manually installed them. I did not bother with resynchronising the host message bus as I had rebooted everything anyway!

## Conclusion
In this post we learnt how to upgrade VMware NSX and the process required to complete an upgrade.  We also worked around the issue of upgrading single host clusters and completed the upgrade without issue.

Until next time :thumbsup:

-Chris