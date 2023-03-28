---
layout: post
title: "Nested Nutanix CE 2.0 Deployment - Part 2" 
excerpt: "Configuration and Test VM Build"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- Nutanix
image:
  thumb: /nested-nutanix-ce-deployment/nested-nutanix-ce-deployment-01.png
comments: true
date: 2023-03-28T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Nutanix and ESXi" src="/images/nested-nutanix-ce-deployment/nested-nutanix-ce-deployment-01.png">
In this post we will complete the basic configuration of our Nutanix AHV installation and deploy a VM into it.

This post is part 2 of a multipart series. Find the other parts here:

- Part 1: [Now it's Time for Something Different](/nested-nutanix-ce-deployment/){:target="_blank"}
- Part 2: This Part: Configuration and Test VM Build

Let's continue.

{% include _toc.html %}
## Configuration
Since part 1 covered purely the deployment of Nutanix CE 2.0, there are some post installation configuration points that we need to cover off prior to deploying our first guest VM later on in this post.

## CVM Certificate 
So that we no longer need to tell chrome **thisisunsafe** each time we access our CVM, let's replace it's SSL certificate.

Click the settings cog in the top right hand corner of the PRISM Elements UI, select **SSL Certificate** from the menu on the left:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Replace Certificate 1" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-16.png">

Select **Replace Certificate > Regenerate Self Signed Certificate** and **Apply**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Replace Certificate 2" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-17.png">

Allow time for the replacement to complete.  That's Chrome happy again!

## Change Default Credentials 
Both our Host and our CVM are using the default credentials:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Default Creds Alarm" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-01.png">

Clicking on each of the alarms in turn provides the following summaries:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Default Creds Host Summary" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-02.png">

And:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Default Creds CVM Summary" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-03.png">

Let's fix that. 

For reference, the Nutanix KB article that covers changing both the Host and CVM default credentials is here: [KB 6153](https://portal.nutanix.com/kb/6153){:target="_blank"}.

## Host Credentials
SSH to the AHV host as root (see [Part 1](/nested-nutanix-ce-deployment/#installing-vmware-tools){:target="_blank"} for default credentials) and issue the following command:
{% highlight shell %}
passwd
{% endhighlight %}

## CVM Credentials
SSH to the CVM as user nutanix (again see [Part 1](/nested-nutanix-ce-deployment/#ahv-vm-config){:target="_blank"} for default credentials)and issue the following command:
{% highlight shell %}
sudo passwd nutanix
{% endhighlight %}

Job done. Clicking on the alters and then clicking **Resolve** clears the alerts from the PRISM Element console.

## Storage Tier Skew on a Node
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Storage Tier Skew on a Node" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-04.png">

From Nutanix [KB 10718](http://portal.nutanix.com/kb/10718){:target="_blank"}:
> The NCC health check plugin node_storage_tier_skew_check reports disk capacity skew above 15% within a storage tier in each individual node in a Nutanix cluster.

As we have only the one host in our cluster, let's turn this check off. Click **Turn Check Off** to complete.

## Cluster Name
Let's name our cluster. Click on the label **Unnamed** and enter a cluster name and virtual IP:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Name Cluster" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-06.png">

Click **Save** when done.

The cluster FQDN needs to resolve to both the cluster virtual IP and to the CVM IP(s). I'm using OPNsense Unbound DNS in my lab, so to achieve this simply add two cluster FQDN host overrides pointing to the virtual and CVM IPs:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OPNsense Unbound site-a-cluster.lab" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-06a.png">

Nice:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="nslookup site-a-cluster.lab" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-06b.png">

## Language  Settings
Select **admin** in the top right hand corner of the PRISM Element UI and select **Update Profile**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Update Profile" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-07.png">

Enter name and email address, hit **Save** when done:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Name + Email Address" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-08.png">

Click the settings cog in the top right hand corner of the PRISM Elements UI and select **Language Configuration**. Set Region accordingly and hit **Save**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Language Settings" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-09.png">

## Networking
Next we will create a network to connect our VMs to. 

Click the settings cog in the top right hand corner of the PRISM Elements UI and select **Network Configuration** and **Subnets**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Subnet Config 1" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-10.png">

Click **Create Subnet**.  

I'll name my subnet **Site-A-Lab**, select vSwitch **vs0** and set my VLAN ID to **0** (native VLAN):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Subnet Config 2" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-11.png">

Click **Save** to complete. Job done:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Subnet Created" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-12.png">

## Upload ISOs
Next, we need to upload our operating system ISO. Whilst we are at it, we will also upload the Nutanix VirtIO for Windows ISO as well (VirtIO ISO can be downloaded from [HERE](https://next.nutanix.com/discussion-forum-14/download-community-edition-38417){:target="_blank"}).

Click the settings cog in the top right hand corner of the PRISM Elements UI, select **Image Configuration** from the menu on the left:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Image Config 1" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-18.png">

Select **Upload Image**. I'll upload my [Windows Server 2022 Evaluation ISO](https://info.microsoft.com/ww-landing-windows-server-2022.html){:target="_blank"}:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Image Config 2" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-19.png">

Allow time for the image to upload.  Repeat for the VirtIO ISO.

Once complete, confirm both ISOs are marked as **ACTIVE**, ready to use:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Image Config 3" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-20.png">
## VM Creation
Let's create our first (not counting the CVM) Nutanix guest VM then! :smile:

From the menu drop down select **VM** and **Create VM**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create VM 1" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-13.png">

I'll configure the following:

* Name: Win-Srv-2022
* Timezone: Europe / London
* vCPUs: 2
* Cores per vCPU: 1
* Memory: 2GB
* Legacy BIOS
* NIC: Connected to Site-A-Lab

Finally, I'll add a 60GB disk to my VM as follows:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create VM 2" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-14.png">

Click **Save** to complete. Select **Table** to view the VM in the list.

## VM Power On and O/S Installation
Click **Power on** to start the VM and **Launch Console** to take a look at it:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Created - Power On" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-15.png">

Once the console of the newly created VM opens, select **Mount ISO**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Mount ISO 1" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-21.png">

Select the O/S ISO uploaded earlier and select **Mount**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Mount ISO 2" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-22.png">

The VM should boot from the ISO:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Boot 1" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-23.png">

I won't cover the full installation of Windows here.  

The only minor ripple to an otherwise normal Windows install was mounting the VirtIO ISO mid windows install to allow Windows to use the VirtIO SCSI controller driver:  

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Boot 2" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-24.png">

Once the driver is loaded, hard disk discovered and selected, change the mounted ISO back to Windows and click **Refresh** within the Windows installer to get it to continue with the installation.

## VirtIO and Nutanix Guest Tools
Upon completion of the Windows installation, mount the VirtIO ISO again and launch the x64 installer:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Launch VirtIO Installer" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-25.png">

Complete the install. 

Back at the PRISM Element VM Table, select **Manage Guest Tools**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Manage Guest Tools" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-26.png">

I'm going to Enable and Mount the guest tools installer. I'll investigate the other options another day:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Enable + Mount NGT" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-27.png">

**Submit** to complete. Run the setup to install:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Install NGT" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-28.png">

## Power Down
Because [UK Energy Crisis](https://www.google.com/search?q=uk+energy+crisis){:target="_blank"}, let's quickly cover safely powering down our Nutanix environment.

### CVM Shutdown
After shutting down all other guest VMs, SSH to the CVM and issue the command:
{% highlight shell %}
cvm_shutdown -P now
{% endhighlight %}

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Shutdown CVM" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-29.png">

### AHV Host Shutdown
Cheat!

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Shutdown AHV Host" src="/images/nested-nutanix-ce-deployment-pt2/nested-nutanix-ce-deployment-pt2-30.png">

That's why we [installed VMTools in part 1](/nested-nutanix-ce-deployment/#installing-vmware-tools){:target="_blank"}. :wink:

## Conclusion and Wrap Up

So there we have it. In this post we built upon the [deployment of our Nutanix AHV host](/nested-nutanix-ce-deployment/){:target="_blank"} by completing it's configuration. 

After that, we deployed a test VM. Finally we looked at how to power the environment down.

Nice!


This post is part 2 of a multipart series. Find the other parts here:

- Part 1: [Now it's Time for Something Different](/nested-nutanix-ce-deployment/){:target="_blank"}
- Part 2: This Part: Configuration and Test VM Build

-Chris