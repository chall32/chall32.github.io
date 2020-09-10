---
layout: post
title: "ESXi 7.0: The Missing PERC(s) - Part 2" 
excerpt: "Re-PERC-ulated"
tags: 
- Free
- Pro-Tip
- VMware
- ESXi
image:
  thumb: esxi7-missing-percs/esxi7-missing-percs-00.png
comments: true
date: 2020-09-10T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Controller" src="/images/esxi7-missing-percs/esxi7-missing-percs-00.png">
Regular readers will know that at in part 3 of the Workaround ESXi CPU Unsupported Error series ([Check the series out](https://polarclouds.co.uk/workaround-esxi-cpu-unsupported/)), during my very limited testing I found that whilst my Dell R710 home server was booting and running ESXi 7.0 quite happily; with just the one exception.... No datastores. 

I followed that discovery up with a tentative - yet ultimately unsuccessful attempt to re-add support for the Dell PERC H700 array controller into ESXi 7.0.  Read about my exploits in [ESXi 7.0: The Missing PERC(s), Lost Control-er](https://polarclouds.co.uk/esxi7-missing-percs/). What follows is an update to that post.

{% include _toc.html %}
## Hopes and Dreams 
At time of writing this post, it has been just over four and a half months since my original missing PERC post and unfortunately - although not really unexpectedly - there has been no release of an ESXi 7.0 compatible native driver for the H700 or LSI 2108 based array controllers in general.

Along with the brilliant feedback from the community (you guys rule :thumbsup:) even with the driver shenanigans, my initial hopes and dreams of LSI 2108 support in ESXi 7.0 where not to come to pass.  Sure, it's the way it goes, older hardware whilst still fully functional ends up obsolete purely due to lack of software support.  

Galling?  Yes. Very.
 
## Upgrade to PERC H710
Yep, so I bit the bullet and invested in a H710 RAID adapter. 

The plan being to get the H710 installed and running under ESXi 6.7 first and then upgrade to ESXi 7.0 later, once the H710 has settled in.

Looking through the [Dell spec sheet](https://www.dell.com/downloads/global/products/pvaul/en/dell-perc-h710p-spec-sheet.pdf), the H710 is based on the LSI SAS2208 chipset. Again, LSI being LSI, the SAS2208 chipset is used in many, many other array controllers.  Here's just a small selection of some of the more popular cards:

- LSI MegaRAID SAS 9265-8i - [pdf](https://docs.broadcom.com/doc/12352136)
- IBM / Lenovo ServeRAID M5110 and M5110e - [pdf](https://lenovopress.com/tips0857.pdf)
- IBM / Lenovo ServeRAID M5120 - [pdf](https://lenovopress.com/tips0858.pdf)
- Fujitsu Server MegaRAID SAS 9286CV-8e SAS -[pdf](https://sp.ts.fujitsu.com/dmsp/Publications/public/ds-py-raid-5-6-SAS-9286CV-8e.pdf)
- Intel RS25DB080 - [pdf](https://www.intel.com/content/dam/www/public/us/en/documents/product-briefs/raid-controller-rs25db080-brief.pdf)
- Intel RMS25PB0x0 / RMS25CB0x0 - [pdf](https://www.intel.com/content/dam/support/us/en/documents/motherboards/server/sb/g37519003_rms25pb080_rms25pb040_rms25cb080_rms25cb.pdf)
- Cisco UCS B420 M3 UCS blade - [pdf](https://www.cisco.com/c/dam/global/en_in/assets/ucs/matrix/r_hcl_B_2-11.pdf)
- HP H210 / H220 HBA - [pdf](https://h20195.www2.hpe.com/v2/getpdf.aspx/c04111455.pdf)
- Dell PERC H810 - [pdf](https://www.dell.com/downloads/global/products/pvaul/en/dell-perc-h810-spec-sheet.pdf)

A more complete list of LSI SAS2208 based controllers is available on the [Serve The Home Forums]( https://forums.servethehome.com/index.php?threads/lsi-raid-controller-and-hba-complete-listing-plus-oem-models.599/post-4319).

The H710 mini is designed to fit into a later 12th generation (PowerEdge Rx20) servers as can be seen from it's proprietary connector:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="H710 Mini" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-01.jpg">

Where as the H710P adapter has a standard PCIe connector:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="H710P" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-02.jpg">

Luckily enough, I was able to pick up my H710P adapter on ebay from a UK seller for £50 including postage. Bargain!

## But Wait There's More
### Bonus 1 - Cabling
As mentioned in a [comment by Mike Vasquez](http://disq.us/p/2ba915j) (cheers Mike!) in an R710 server, the H700 cables work just fine with the H710. Yep they sure do:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="H710 Fitment" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-03.jpg">

### Bonus 2 - ESXi 7.0 Driver
Checking the [VMware Compatibility Guide for the H710 adapter](https://www.vmware.com/resources/compatibility/detail.php?deviceCategory=io&productid=21816&vcl=true), surprise upon surprise there is a separate ESXi 7.0 driver available!!!  [Check it out here](https://my.vmware.com/group/vmware/downloads/details?downloadGroup=DT-ESXI65-AVAGO-DELL-SHARED-PERC8-068069000-1OEM&productId=614&download=true&fileId=121def8bb0401e05ad993f36f7ce0343&uuId=0ea62858-446c-4924-bc08-29d0739edc31).

Strangely the release notes for the driver talk about ESXi 6.5 support:

> VMware ESXi 6.5 GA support for the Dell Shared Perc H710 Adapter 6Gbps Family of SAS controllers<br> 
Driver name and version:dell-shared-perc8-06.806.90.00-1OEM.650.0.0.4598673<br>
Compatible ESX version: VMware ESXi 6.5<br>
Controller Firmware Package:23.14.06.0013<br>
Dependencies: None<br>
Bugs fixed (compared to earlier release of driver): Fixes a potential issue where in a clustered configuration snapshot, VMotion, and cloning operations could fail when writing to shared storage.<br>
Known Issues and Workarounds: None<br>
Additional configuration options supported by the driver: None<br>

Having said that, I think I'll stick with the "in box" lsi-mr3 ESXi 7.0 native driver.

...but I'm getting ahead of myself.  

## Upgrading From H700 to H710
Let's talk about the worrying part of all of this: is it possible to upgrade from a H700 to a H710 **WITHOUT DATA LOSS**? 

Will an LSI 2208 controller read, accept and run with an array created by an LSI 2108 controller?  Let's find out!

### Backup, Backup, Backup
Yep, I spent the best part of a day backing up all of my data.  Finding out the hard way that such an upgrade is indeed not possible was at the forefront of my day spent preparing for the upgrade.

### Removing the H700 and Fitting the H710
Simple enough, power off the server, unplug and remove the H700 array controller and plug in the H710 array controller.

As the H710 array controller isn't "an official option" for the PowerEdge R710 server, chances are that it won't work in the R710's dedicated storage adapter slot. I didn't bother taking the time to find out, I simply plugged my H710 controller into PCI slot 2 as can be seen from the picture above.

Plug in the SAS cables and it's job done.

### First Power On
Let's power on...

Remember, the plan is to get the H710 installed and running under ESXi 6.7 first and then upgrade to ESXi 7.0 later, once the H710 has settled in.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 1" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-04.png">

OK, so H710 initialising OK.  Looks like firmware is latest version too, version [21.3.5-0002](https://www.dell.com/support/home/en-uk/drivers/driversdetails?driverid=9mhj5).

Lets quickly press **F** to import the foreign configuration (to the controller) from the disks:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 2" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-05.png">

Hmmm, slightly more worrying... 

> All of the disks from your previous configuration are gone

Lets hope it's talking about the configuration on the controller... :worried:

Press **Y** to continue and make configuration changes and hold my breath:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 3" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-06.png">

:boom:**BOOM!**:boom: Two RAID5 sets, six disks... **HORRAY!!!** :grin:

Right let's look further. Yep, six online disks:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 4" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-07.png">

Controller BIOS is enabled.  I'm booting from USB anyway, so not a big deal:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 5" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-08.png">

Yep, firmware up to date and card fitted into slot 2:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 6" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-09.png">

Let's exit out of the H710 configuration utility and continue to boot back into ESXi 6.7:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 7" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-10.png">

Once ESXi boots, lets take a look at the ESXi host Web client. Yep H710 recognised: 

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 8" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-11.png">

What's more is that it's using the in box ESXi 6.7 lsi_mr3 driver:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 9" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-12.png">

Checking using the [perccli add in](https://www.dell.com/support/article/en-uk/sln283135/how-to-use-the-poweredge-raid-controller-perc-command-line-interface-cli-utility-to-manage-your-raid-controller?lang=en), looks good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 10" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-13.png">

Finally, checking via Dell OpenManage, looks like the array controller is re-initialising the arrays in the background:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power On 11" src="/images/esxi7-missing-percs-pt2/esxi7-missing-percs2-14.png">

Lets leave bring all of the VMs back online and leave the background initialisation to continue in the er, background. :grin:

All in all, it took just under 12 hours to finish the background initialisation of both RAID5 sets.

## Conclusion and Wrap Up
So there we have it. From this we have learned that:

- a PERC H710 will import and run with a PERC H700 created RAID set quite happily<br>
or<br>
- an LSI 2208 chipset will run with an LSI 2108 created RAID set quite happily<br>
or<br>
- an LSI MegaRAID SAS 9265 run with an LSILSI MegaRAID SAS 9260 created RAID set quite happily

Potentially you could cross vendors too - say replace an Lenovo / IBM ServeRAID M5015 with a Intel RS25DB080... well, you get my drift.  

Anyway, I'll let this settle in a little further after which it's ESXi 7.0 time.  Don't worry, rest assured, I'll post here when I complete the upgrade. :wink:

-Chris