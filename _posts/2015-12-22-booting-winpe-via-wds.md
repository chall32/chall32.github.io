---
layout: post
title: Booting Windows Preinstallation Environment Diagnostics from Windows Deployment Services Server
excerpt: Loading WinPE based diagnostic tools as quickly as possible...
tags:
- Deployment
- Windows
image:
  thumb: booting-winpe-via-wds/wdswinpe00.png
comments: true
date: 2015-12-22T20:15:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Windows 2012R2 Logo" src="/images/booting-winpe-via-wds/wdswinpe00.png">
Previously we configured a Windows 2012R2 Windows Deployment Services (WDS) Server to deploy Windows installations over the network. In case you missed it, have a read [here](/windows-2012r2-wds/) to see what we did.

Next, we upgraded our WDS server setup to also deploy VMware ESXi. Again have a read [here](/deploying-vmware-esxi-via-wds/) to see what we did next. 

Now it's time use our WDS server to allow us to quickly boot some diagnostic tools. 

This time: Windows Preinstallation Environment (WinPE).

{% include _toc.html %}

### Windows Preinstallation Environment Preamble
For those unfamiliar with it, Windows Preinstallation Environment (WinPE) is a lightweight version of Windows used for the deployment of PCs, workstations, and servers, or troubleshooting an operating system while it is offline. More info [here](https://en.wikipedia.org/wiki/Windows_Preinstallation_Environment).

For this, I'll use my (now ancient) Windows 7 Preinstallation Environment (Win7PE) image I built with [WinBuilder](http://winbuilder.net/) back in 2009.  I'm not going to cover the creation of a Windows PE image here as there are plenty of guides out there that can show you how to create your own Win7PE image.

One such guide is here:
<iframe width="560" height="315" src="https://www.youtube.com/embed/2vCyIIqkeiM"> </iframe>
<br>
Although WinPE is described as being "lightweight" you can still end up with quite a hefty image to boot from.  Certainly my old Win7PE ISO image weighs in at just over 370MB.  Because of this, booting WinPE from CD/DVD is not quick.  

### Adding Win7PE to WDS Server
Lets speed up the time it takes for us to boot a Win7PE diagnostic CD/DVD.

On you WDS Server, open Windows Deployment Services tool (Start > Administrative Tools > Windows Deployment Services), expand the tree, right click Boot Images and select "Add Boot Image":
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 1" src="/images/booting-winpe-via-wds/wdswinpe01.png">

Insert / mount your Win7PE CD/DVD. Browse to the sources folder on the CD and click next:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 2" src="/images/booting-winpe-via-wds/wdswinpe02.png">

Enter a name and description for your Win7PE boot image:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 3" src="/images/booting-winpe-via-wds/wdswinpe03.png">

Click Next if all looks OK:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 4" src="/images/booting-winpe-via-wds/wdswinpe04.png">

Ahh a problem...
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 5" src="/images/booting-winpe-via-wds/wdswinpe05.png">

The fix for the error "The boot files for this architecture are not installed on the server" is detailed [below](#fixing-the-boot-files-for-this-architecture-are-not-installed-on-the-server-error).

Try again...
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 6" src="/images/booting-winpe-via-wds/wdswinpe06.png">

Done.  A quick double check:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 7" src="/images/booting-winpe-via-wds/wdswinpe07.png">

Looks good.

### Booting WinPE from WDS Server
PXE boot your machine as normal.  Choose "Windows Deployment Services Boot" (feel free change the menu item to something like "Windows Deployment Services / Windows Preinstallation Boot" if you like):
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 8" src="/images/booting-winpe-via-wds/wdswinpe08.png">

Select Win7PE:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 9" src="/images/booting-winpe-via-wds/wdswinpe09.png">

Give it a minute:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 10" src="/images/booting-winpe-via-wds/wdswinpe10.png">

Job's a good-un:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Add Win7PE to WDS 11" src="/images/booting-winpe-via-wds/wdswinpe11.png">

Well there you have it, network booting Windows Preinstallation Environment based diagnostics. Sooo much quicker to boot than a disc of spinning plastic! (CD/DVD etc)

Next time Acronis Disk Director and other tools.

-Chris

<br>

<br>

<br>

### Fixing "The boot files for this architecture are not installed on the server" Error
What looks like quite a catastrophic error is actually quite easily fixed.  Remember where we added our PXElinux files to our WDS sever [here](/deploying-vmware-esxi-via-wds/#step-1-wds-client-boot-image)?
Well, we need to temporarily back the boot file change out thus allowing us to add boot images to our WDS server. 

Open explorer and browse to C:\RemoteInstall\Boot\x86.

Find the file pxeboot.0 and copy it to a new file called pxeboot.n12.

Next run the following from an administrative command prompt:
{% highlight text %}
wdsutil /set-server /bootprogram:boot\x86\pxeboot.n12 /architecture:x86
wdsutil /set-server /N12bootprogram:boot\x86\pxeboot.n12/architecture:x86
net stop wdsserver
net start wdsserver 
{% endhighlight %}

Have another go at adding [Win7PE to WDS](#adding-win7pe-to-wds-server)

Once complete, don't forget to swap the boot files back:
{% highlight text %}
wdsutil /set-server /bootprogram:boot\x64\pxelinux.0 /architecture:x64
wdsutil /set-server /N12bootprogram:boot\x64\pxelinux.0 /architecture:x64
net stop wdsserver
net start wdsserver
{% endhighlight %}

<br>
