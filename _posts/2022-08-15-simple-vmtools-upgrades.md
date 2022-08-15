---
layout: post
title: "Simple VMTools Upgrades - The Lazy Guide" 
excerpt: "Never Used to be a Thing... It is Now"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
image:
  thumb: /vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-01.png
comments: true
date: 2022-08-15T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-01.png">
*In observance of [National Lazy Day 2022](https://www.wincalendar.com/uk/Lazy-Day){:target="_blank"}... Yeah I know, it was 5 days ago... But hey, who's counting?*

Back when, VMtools used to be updated with each ESXi patch installation.  Just patch ESXi, an updated VMTools would be rolled in and away we go; VMs to update.

Not any more.

So what is the simplest way to handle VMTools updates?

Follows is the **Lazy Guide**.<br>

***Not that VM admins are lazy - far from it!** VM admins have other things to do; Networking with [NSX-T](https://www.vmware.com/uk/products/nsx.html){:target="_blank"}, Storage with [vSAN](https://www.vmware.com/uk/products/vsan.html){:target="_blank"}, Compliance with [vRO](https://www.vmware.com/uk/products/vrealize-operations.html){:target="_blank"}, Cloud Management with [vCD](https://www.vmware.com/uk/products/cloud-director.html){:target="_blank"}, Disaster Recovery with [SRM](https://www.vmware.com/uk/products/site-recovery-manager.html){:target="_blank"} to name a few. You know, other stuff to be getting on with...* :wink:
{% include _toc.html %}
## Overview
Other methods of updating VMtools are available such as the [VMTools Repository Method](https://blogs.vmware.com/vsphere/2019/01/configure-a-vmware-tools-repo-in-vsphere-6-7u1.html){:target="_blank"}, the [Manual Method](https://docs.vmware.com/en/VMware-Tools/12.0.0/com.vmware.vsphere.vmwaretools.doc/GUID-B632D26F-410A-43C9-9BFD-21EBB21DE397.html){:target="_blank"} and the [SCCM method](https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/techpaper/deploying-vmware-tools-using-sccm-user-guide.pdf){:target="_blank"}.

Whilst those methods are okay, they do require either "getting down and dirty" with the vCenter/ESXi API (the repository method), lots of intervention (the manual method) or a System Center Configuration Manager (the SCCM method).

We are going to go simplicity. This method requires two steps:

1. Update VMtools package via a baseline profile / update 
2. Configure VMs to auto update when a new version of VMtools is detected

As a bonus, we will look at achieving the above in both an enterprise deployment (I.E with vCenter Server(s)) and a smaller single host / stand-alone environment, without vCenter.

Finally, this post will take the form of a "beginners guide" with extra hand-holding and screenshots for those new to vSphere or unfamiliar with updating VMTools using these methods.
## Download Latest VMTools
The permanent link to download VMtools is [https://www.vmware.com/go/tools](https://www.vmware.com/go/tools){:target="_blank"}. For both with and without vCenter methods below, we require the **VMware Tools Offline VIB Bundle**.

With latest VMtools VIB in hand, lets get to it.

## With vCenter Method
*TL;DR - Import VIB, create a baseline, attach and remediate.*

### Import VIB + Create Baseline
From vCenter client (https://vcenter.fqdn/ui/), select **Menu > Lifecycle Manager**:
 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Open Lifecycle Manager" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-02.png">
Select **Actions > Import Updates**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Import Updates" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-03.png">

Select previously downloaded VMtools VIB and allow to upload:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Upload Update" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-04.png">

Select **Baselines > New > Baseline**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Baseline" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-05.png">

Enter Baseline name, description and select **Patch**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Baseline Details 1" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-06.png">

Click **Next**.

Untick "Automatically update this baseline with patches that match the following criteria" and click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Baseline Details 2" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-07.png">

Deselect "Show only rollup updates" and filter name on **tools**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Baseline Details 3" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-08.png">

Select the version VMtools that matches your newly uploaded VMTools and ESXi versions. In my case below, I'm selecting VMTools version 12.0.6 and ESXi 7.0.*:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Baseline Details 4" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-09.png">

Click **Next**, confirm all looks correct and finally click **Finish**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Baseline Details 5" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-10.png">

Yep all looks good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="New Baseline Details 6" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-10a.png">

### Attach Baseline and Update Hosts
Select **Menu > Inventory**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Open Inventory" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-11.png">

As we want ALL ESXi hosts in our site to have the updated VMTools package, we will attach the baseline at our site level. Select your datacenter (in my case "SITE-A"), **Updates** and **Baselines**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site Baseline 1" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-12.png">

Scroll down and select **Attach Baseline or Baseline Group**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site Baseline 2" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-13.png">

Select the previously created VMTools baseline and click **Attach**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Site Baseline 3" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-14.png">

From the **Inventory > Datacenter > Updates > Baselines** view, select **Check Compliance**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 1" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-15.png">

Once the compliance check has completed, you will see that your hosts will be flagged as "Non-compliant":

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 2" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-16.png">

Scroll down and select **Pre-check Remediation**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 3" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-17.png">

Confirm that there are no issues flagged that may prevent the completion of remediation:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 4" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-18.png">

Click **Done**. 

Scroll down further, select the VMTools package attached earlier and finally select **Remediate**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 5" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-19.png">

Again, confirm all looks goo and click **Remediate** again:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 6" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-20.png">

Monitor the installation via Recent Tasks:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 7" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-21.png">

Job Done:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 8" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-22.png">

Giving ESXi ten minutes to catch up, yep our Windows VMs need updating:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Remediate Hosts 9" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-23.png">

Don't worry, I have a lazy way to handle those too. :wink:
## Stand-alone ESXi (Without vCenter) Method
This method is a little simpler.

First off, we need to open our downloaded VMTools zip file and extract a VIB. Opening the zip and navigating into the **vib20\tools-light** folder, lets extract our VIB:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 1" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-24.png">


Log onto the ESXi management interface (https://esxi.fqdn/ui/), select **Storage** and a Datastore. In my case I selected "LOCAL1":

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 2" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-25.png">

Make a note of the location. This should be easily copied:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 3" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-26.png">

Select **Datastore browser** and **Upload**. Select the VIB extracted earlier and click **Open**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 4" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-27.png">

Once the upload has completed, select **Manage > Packages > Install Update**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 5" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-28.png">

Using the location noted earlier, add the name of the VIB file, paste into the install update dialogue and select **Update**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 6" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-29.png">

Click **Continue**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 7" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-30.png">

Monitor and confirm successful completion:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 8" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-31.png">

Finally, refresh the packages page, search for "tools-light" and confirm the updated version is listed:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 9" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-32.png">

Again, giving ESXi ten minutes to catch up, yep our Windows VMs need updating:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi Update 10" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-33.png">

## Installing VMTools in VMs
The simplest way to handle updating of VMTools in VMs is two-fold:

### Linux and FreeBSD VMs
Use Open VM Tools for Linux and FreeBSD VMs rather than the VMware supplied VMTools. Open VM Tools are bundled with Linux and FreedBSD and are updated via distribution updates, therefore do not need updating separately. See [Open VM Tools](https://docs.vmware.com/en/VMware-Tools/12.0.0/com.vmware.vsphere.vmwaretools.doc/GUID-8B6EA5B7-453B-48AA-92E5-DB7F061341D1.html){:target="_blank"} for details.

### Windows VMs
Use automation. Lets look at that next.

## Automating VMTools Updates (GUI Method - With vCenter)
If we dig into our Windows VMs and take a look at the **Updates** tab, we can see the following option to update VMtools automatically when a VM is rebooted:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Update 1" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-34.png">

Since we are talking Windows based VMs here, we know that through normal maintenance patching, our VMs are rebooted *at least* once a month. 

Lets configure auto VMTools auto update then.  Select **Site > VMware Tools**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Update 2" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-35.png">

After selecting the Cluster and our Windows VMs, we can set auto update to **On**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Update 3" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-36.png">

Job done.

## Automating VMTools Updates (GUI Method - Without vCenter)
Log into ESXi web client, select a Windows VM, **Edit > VM Options**, expand **VMware Tools** and enable **Check and upgrade VMware Tools before each power on**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Update 4" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-37.png">

Save and repeat for remaining Windows VMs.

## Automating VMTools Updates (Script Method - Both With and Without vCenter)
The above methods VMTools Update methods are acceptable small scale, but what happens if we have hundreds of VMs to deal with?

Let's use PowerShell! :sunglasses:

First let's create a report of all Windows VMs that are not currently set to auto update VMTools:
<figure><figcaption><b>Filename: </b>Report-VMToolsAutoUpgrade.ps1</figcaption> 
{% highlight shell linenos %}
$VC = @("vcenter.local")
$StoredCred = "vSphere-Admin"
$Report = "C:\Scripts\VM-Tools\V2\Enable-VMToolsAutoUpgrade-Report.csv"
$Credentials = Get-StoredCredential -Target $StoredCred
Connect-VIServer -Server $VC -Credential $Credentials
$UPGNeeded = Get-VM | Where {$_.ExtensionData.Config.Tools.ToolsInstallType -eq "guestToolsTypeMSI" -and `
                             $_.ExtensionData.Config.Tools.ToolsUpgradePolicy -like "manual" -and `
                             $_.ExtensionData.Config.ManagedBy.type -ne "placeholderVm" -and `
                             $_.Name -notlike "*FRED*"}
$UPGNeeded | Select-Object `
             Name,`
             PowerState,`
             @{label='Current Tools Version'; expression={($_.ExtensionData.Guest.ToolsVersion)}}, `
             @{label='Operating System'; expression={($_.Guest.OSFullName)}}, `
             @{label='VM Notes'; expression={($_.Notes[0..51] -join "")}} `
             | Export-Csv -Path "$Report" -NoTypeInformation
Write-Host "`n Report saved to $Report"
Disconnect-VIServer -Server * -Confirm:$false 
{% endhighlight %}
</figure>
Breaking the report script down:

- **Lines 1 to 5**: Configure and connect to vCenter / ESXi using Credential Manager as discussed in [PowerShell Credential Handling](/powershell-credential-handling/){:target="_blank"} post.
- **Lines 6 to 9**: Select VMs where VMTools install like "guestToolsTypeMSI" (IE Windows VMs), where upgrade policy is like "manual", where VM is not a SRM place holder VM and finally with a VM name not like "FRED".
- **Lines 10 to 16**: From the VMs selected, export name, power state, current tools version, guest operating system and VM notes (truncated to 50 characters).
- **Line 17**: Export .csv report 
- **Line 18**: Disconnect from vCenter / ESXi

The above will create a .csv spreadsheet containing following information:

|Name  |PowerState	|Current Tools Version | Operating System | VM Notes |
|:----:|:-----------|:-----|:-----------------------|:-----------------|
|APP1  |PoweredOn	|11334	|Microsoft Windows Server 2016 (64-bit)	|Test Server APP1|
|APP2  |PoweredOn	|11334	|Microsoft Windows Server 2016 (64-bit)	|Test Server APP2|

Notice that the **VM-CH-ESG-SITE-A** is not listed in the report?

That's because it's a Linux VM running with Guest Managed VM tools:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Linux VM" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-38.png">

After confirming the .csv spreadsheet contains the correct data, the next step is to enable VM tools on the applicable VMs.  Again, we will use a script for this:

<figure><figcaption><b>Filename: </b>Enable-VMToolsAutoUpgrade.ps1</figcaption> 
{% highlight shell linenos %}
$VC = @("vcenter.local") 
$StoredCred = "vSphere-Admin"
$Report = "C:\Scripts\VM-Tools\V2\Enable-VMToolsAutoUpgrade-Report.csv"
$Credentials = Get-StoredCredential -Target $StoredCred
Connect-VIServer -Server $VC -Credential $Credentials
$VMs = Import-CSV "$Report"
$vmConfigSpec = New-Object VMware.Vim.VirtualMachineConfigSpec
$vmConfigSpec.Tools = New-Object VMware.Vim.ToolsConfigInfo
$vmConfigSpec.Tools.ToolsUpgradePolicy = "UpgradeAtPowerCycle"
ForEach($VM in $VMs){ 
    Get-VM -Name $($VM.Name) | ForEach {$_.ExtensionData.ReconfigVM_task($vmConfigSpec) > $null}
    Write-Host "Reconfigured $($VM.Name)"
}
Disconnect-VIServer -Server * -Confirm:$false 
{% endhighlight %}
</figure>
Breaking the enable script down :

- **Lines 1 to 5**: Configure and connect to vCenter / ESXi using Credential Manager as discussed in [PowerShell Credential Handling](/powershell-credential-handling/){:target="_blank"} post.
- **Line 6**: Import the report .csv
- **Lines 7 to 9**: Create the VM [configuration specification](https://vdc-repo.vmware.com/vmwb-repository/dcr-public/1ef6c336-7bef-477d-b9bb-caa1767d7e30/82521f49-9d9a-42b7-b19b-9e6cd9b30db1/vim.vm.ConfigSpec.html){:target="_blank"} 
- **Line 10 to 13**: Loop through the .csv report and apply the configuration specification to each VM listed and feedback 
- **Line 14**: Disconnect from vCenter / ESXi

Let's run the script:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Enabling VMTools Auto Update" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-39.png">

Looks good.  Checking one of the VMs reconfigured:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VMTools Auto Update Enabled" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-40.png">

Boom!

After a reboot of the VM:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VMTools Auto Updating" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-41.png">

Something to be aware of, sometimes VMs require a reboot to fully install VMTools:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VMTools Auto Reboot" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-42.png">

If you are rebooting your VMs in a maintenance window, what difference does second reboot make anyway?

And we are done!

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VMTools Upgrade Done" src="/images/vmtools-upgrades-lazy-guide/vmtools-upgrades-lazy-guide-43.png">

## Conclusion and Wrap Up
So there we have it. A simple way to update VMTools using Baselines and Lifecycle Manager in vCenter if you have it, or via ESXi host VIB update if not.

We then looked at using the Open-VM-Tools package for Linux and FreeBSD based VMs.

Finally we looked at manual and scripted methods to configure VMs to auto update their VMTools installations when they are rebooted - say during a maintenance window. 

All with ease. Happy belated Lazy Day 2022!

-Chris