---
layout: post
title: "Simple VMware vSphere Estate Patching and Updating"
excerpt: Two downloads and two commands 
tags:
- Free
- VMware
- ESXi
- Pro-Tip
image:
  thumb: vsphere-patch-update/vsphere-67u3-00.png
comments: true
date: 2019-08-26T16:00:00+00:00
---
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Patch and Update" src="/images/vsphere-patch-update/vsphere-67u3-01.png">
<span class="image-credit" style="float: right; margin: 0px 0px 0px 10px;">Photo: <a href="https://unsplash.com/@randyfath?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Randy Faith</a></span>

Another month another set of vSphere patches. This month, VMware have just released vSphere 6.7 Update 3 for ESXi and vCenter. 

It occurred to me that I hadn't previously detailed how I go about patching vSphere environments including my NSX-V Lab.  To rectify that, what follows is the method I use to patch and update vSphere.

Yes, there are other ways to achieve the same outcome using update manager etc. Using this method, we simply grab the updates from the VMware update site, push them to the correct locations and install.

What follows is a whistle-stop guide to update a vSphere environment. Whilst it is by no means exhaustive, the method detailed here:

   :heavy_check_mark: Can be employed in an isolated environment  
   :heavy_check_mark: Is fully supported by VMware  
   :heavy_check_mark: Simple!  
   :heavy_check_mark: Serves as a reminder to me - for the next time :wink:  

It goes without saying (although I'm saying it here!), that this guide is written with the target audience being the experienced vSphere administrator. 

{% include _toc.html %}
## Confirming Compatibility
As previously mentioned, I need to patch my NSX lab.  As NSX runs as a plug-in to vCenter, I need to confirm that updating vCenter won’t cause NSX any issues. Other VMware products such as Site Recovery Manager plug-in to vCenter in the same way.

The version of NSX I have running in my lab is 6.4.5. I need to confirm that this version is compatible with my target vSphere version; in this case vCenter 6.7 update 3 and ESXi 6.7 update 3.

Lets have a look at the [VMware Product Interoperability Matrices](https://www.vmware.com/resources/compatibility/sim/interop_matrix.php)

Plumbing in *NSX for vSphere 6.4.5* in section 1, *VMware vCenter Server 6.7 U3* and *VMware vSphere Hypervisor (ESXi) 6.7 U3* into the tool produced the following results:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX compatibility check" src="/images/vsphere-patch-update/vsphere-67u3-02.png">

All looks good.

## Updating vCenter
Task one is to update vCenter.  The golden rule when applying updates to vSphere environments: **vCenter must be of equal or greater version to the ESXi hosts it manages**.

Head over to [VMware patch download site](https://my.vmware.com/group/vmware/patch) and grab the vCenter update bundle iso file.  This is the one I need:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="vCenter update patch" src="/images/vsphere-patch-update/vsphere-67u3-03.png">

Next, we need to mount the downloaded iso file onto one of the lab vCenter servers.  Simplest way is to open a console to the vCenter server from the ESXi management interface and mount the iso to the CD/DVD drive using the VMRC client:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Mount vC update iso file" src="/images/vsphere-patch-update/vsphere-67u3-04.png">

Then we need to login as root to our vCenter via `https://<vCenter FQDN or IP Address>:5480` Select **Update - Check Updates - Check CD ROM**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Check CD ROM for update" src="/images/vsphere-patch-update/vsphere-67u3-05.png">

Open the "twistie" and click **Run pre-update checks**. Once complete, click **Stage and Install**.  Complete the licence, CEIP and backup confirmation wizard and hit finish.  The update will now install:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="vC update installing" src="/images/vsphere-patch-update/vsphere-67u3-06.png">

Keep an eye on the vCenter console via VMRC.  Once the update installation has completed, the vCenter VM should reboot.

Once vCenter has booted, log on and confirm all working OK, including NSX and any other vCenter plugins you may have.

Repeat update process for any other vCenter servers in your environemnt.

## Updating ESXi
Head over again to [VMware patch download site](https://my.vmware.com/group/vmware/patch) and grab the required ESXi update zip file.  This is the one I need:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi update patch" src="/images/vsphere-patch-update/vsphere-67u3-07.png">

Upload the update zip file to your ESXi host using the datastore browser:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Upload patch to datastore" src="/images/vsphere-patch-update/vsphere-67u3-08.png">

Enable SSH, connect to the host and login as root.

Run the command `esxcli software sources profile list -d </path/to/patch zip>` to list the image profiles contained within the update:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="List update profiles" src="/images/vsphere-patch-update/vsphere-67u3-09.png">

As you can see from the above, the update includes four profiles:

1. `ESXi-<update version>s-no-tools` - Contains security patches only with no VMware Tools 
2. `ESXi-<update version>-no-tools` - Contains all patches with no VMware Tools
3. `ESXi-<update version>-standard` - Contains all patches and VMware Tools
4. `ESXi-<update version>s-standard`  - Contains security patches only with VMware Tools  

As I'm going to install all patches and update VMware tools, I'm going to opt for profile 3 **ESXi-6.7.0-20190802001-standard**

Use the command `esxcli software profile update -d </path/to/patch> -p <image profile>` to install the update:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Install ESXi update" src="/images/vsphere-patch-update/vsphere-67u3-10.png">

Yes, my lab is a little old:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Hardware warning" src="/images/vsphere-patch-update/vsphere-67u3-11.png">

Lets try again: `esxcli software profile update -d </path/to/patch>`<br>`-p <image profile> --no-hardware-warning`

<img style="display: block; margin-left: auto; margin-right: auto;" alt="ESXi update installed" src="/images/vsphere-patch-update/vsphere-67u3-12.png">

That's better.  Reboot time!

Boom! Done:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Done" src="/images/vsphere-patch-update/vsphere-67u3-13.png">

On to patching my other lab host! :smile:

Happy patching.

-Chris



 

