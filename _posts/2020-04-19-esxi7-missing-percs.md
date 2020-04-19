---
layout: post
title: "ESXi 7.0: The Missing PERC(s)" 
excerpt: "Lost Control-er"
tags: 
- Free
- Pro-Tip
- VMware
- ESXi
image:
  thumb: esxi7-missing-percs/esxi7-missing-percs-00.png
comments: true
date: 2020-04-19T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Controller" src="/images/esxi7-missing-percs/esxi7-missing-percs-00.png">
Regular readers will know that at in part 3 of the Workaround ESXi CPU Unsupported Error series ([Check them out](https://polarclouds.co.uk/workaround-esxi-cpu-unsupported/)), during my very limited testing I found that whilst my Dell R710 home server was booting and running ESXi 7.0 quite happily; with just the one exception.... No datastores. 

My Dell PERC H700 RAID controller had gone AWOL under ESXi 7.0.

In this post, I'll try to address that. First off the standard disclaimer applies:

-  Proceed at your own risk
-  What follows is totally unsupported
-  You alone are responsible for the servers in your care
-  These modifications should NOT be made on production systems

Also - whilst I've tested what follows in a VM,<br>
**I HAVE NOT YET TESTED WITH REAL HARDWARE!**
{: style="color:red; text-align: center;"}

{% include _toc.html %}

## VMKlinux Driver Stack Deprecation
After some research, I found this post on the VMware blogs site: [What is the Impact of the VMKlinux Driver Stack Deprecation?](https://blogs.vmware.com/vsphere/2019/04/what-is-the-impact-of-the-vmklinux-driver-stack-deprecation.html)<br>
As you can guess from that title, from v7.0 onwards, ESXi will no longer support VMKlinux based drivers.

Hmm, OK. I kind of know the answer already, but lets see. My R710 is currently running ESXi 6.7U3, so lets run the suggested test:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Dell H700 + H800" src="/images/esxi7-missing-percs/esxi7-missing-percs-02.png">

Yeah, no surprises there. We did the testing the hard way anyway. It's the H700 that's using a VMKlinux driver. 

We can confirm that by looking in the VMware Compatibility Guide (VCG):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="H700 HCL" src="/images/esxi7-missing-percs/esxi7-missing-percs-03.png">

Time to do some reading...

## Dell PERC H700 RAID Controller - Tell Me More
According to Dell's own documentation the [PERC Technical Guidebook](https://www.dell.com/downloads/global/products/pvaul/en/perc-technical-guidebook.pdf) lists the H700 and H800 adapters as based on the LSI 2108 chipset:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Dell H700 + H800" src="/images/esxi7-missing-percs/esxi7-missing-percs-01.png">

We know that LSI don't just make chipsets for Dell.  What other RAID controller cards use the LSI 2108 chipset? 

-  LSI MegaRAID SAS 9260 / 9260DE - [pdf](https://www.starline.de/fileadmin/images/produkte/lsi/LSI_MR_SAS9260-8i__ENG_.pdf)
-  Lenovo / IBM ServeRAID M5015 / M5014 - [pdf](https://lenovopress.com/tips0738.pdf)
-  Intel RS2BL080 / RS2MB044 - [pdf](https://www.intel.com/content/dam/support/us/en/documents/motherboards/server/rs2bl080/sb/e64388004_rs2bl080_rs2mb044_tps_21.pdf)
-  Fujitsu S26361-D2616-Ax / S26361-D3016-Ax - [pdf](http://manuals.ts.fujitsu.com/file/3490/lsi-modular-raid-ug-en.pdf)
- Cisco C200 / C460 / B440 Servers - [pdf](https://www.cisco.com/c/en/us/td/docs/unified_computing/ucs/c/sw/raid/configuration/guide/RAID_GUIDE.pdf)

...to name just a few.

I even found the [LSI 2015 Product Guide](https://www.exertishammer.com/assets/uploads/resources/Broadcom%20-%20Product%20Brochure%20-%20Avago%20Storage%20Solutions%20Product%20Guide.pdf) that contains something like thirteen LSI 2108 chipset based cards if you fancy spending a couple of hours on VMware Compatibility Guide!!

## Clutching at Straws
Checking the VCG with the first five above, straight away I noticed something: 
 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Native and vmlinx" src="/images/esxi7-missing-percs/esxi7-missing-percs-04.png">
 
Hmmm, so back in the ESXi 5.5 days, LSI 2108 based cards used the native lsi_mr3 driver...!?! Check out the above VGC entry for the LSI MegaRAID SAS 9260-8i for yourself [HERE](https://www.vmware.com/resources/compatibility/detail.php?deviceCategory=io&productid=12384)

Wait, when did the VMKlinux Driver Stack Deprecation start? [ESXi5.5!](https://www.virtuallyghetto.com/2013/10/esxi-55-introduces-new-native-device.html)

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Magic!" src="/images/esxi7-missing-percs/magic.gif">

## Adding H700 Support Back to lsi_mr3
Many modern operating systems use PCI card hardware identifiers (aka PCI IDs) to identify and to ensure that the correct driver is loaded for the hardware present. Taking the H700 for example, we can see from the VMware VCG (after pulling all the entries together):

Vendor ID (VID) = 1000<br>
Device ID (DID) = 0079<br>
SubVendor ID (SVID) = 1028<br>
SubDevice ID (SDID) = 1f16 (Dell PERC H700 Adapter)<br>
SubDevice ID (SDID) = 1f17 (Dell PERC H700 Integrated)<br>
SubDevice ID (SDID) = 1f18 (Dell PERC H700 Modular)<br>

After lots and lots of reading, a bit more reading and a bit of testing in a VM, it looks like VMware drivers potentially reference PCI hardware IDs located in two files for each driver present in the O/S.  These files are `driver.map` and `driver.ids`.

What happens if we add the PCI ID of the Dell H700 to the list of IDs supported by the lsi_mr3 driver?  After all, if the VGC is anything to go by, the lsi_mr3 driver *used* to support LSI 2008 based cards...

So using my ESXi 7 VM [created earlier](https://polarclouds.co.uk/workaround-esxi-cpu-unsupported-pt3/#create-a-vm-for-esxi-installation-on-usb), lets have a play. As ESXi runs from memory, we need to extract lsi_mr3.v00, make the required modifications and repackage the modified files back into lsi_mr3.v00. Finally reboot to load the modified driver. Follows is the process to extract, modify and repackage the lsi_mr3.v00 driver.

In this example we are using a datastore called "datastaore1" yours maybe named differently. I've possibly gone a bit overboard with the following instructions, but hey, better to much than to little. 

1. Enable SSH on your ESXi host
2. Connect with ssh client (eg putty)
3. Copy the compressed driver to somewhere where we can work on it:<br> `cp /tardisks/lsi_mr3.v00 /vmfs/volumes/datastore1/s.tar`
4. Move to to datastore:<br> `cd /vmfs/volumes/datastore1/`
5. Extract one:<br>`vmtar -x s.tar -o output.tar`
6. Clean up:<br>`rm s.tar`
7. Create a temp working directory:<br>`mkdir tmp`
8. Move our working file to the temp directory:<br>`mv output.tar tmp/output.tar`
9. Move the the temp directory:<br>`cd tmp`
10. Extract two:<br>`tar xf output.tar`
11. Clean up:<br>`rm output.tar`
12. Edit the map file:<br>`vi /vmfs/volumes/datastore1/tmp/etc/vmware/default.map.d/lsi_mr3.map`<br>See [KB1020302](https://kb.vmware.com/s/article/1020302) for help with vi
13. Paste in the following:<br>`regtype=native,bus=pci,id=10000079..............,driver=lsi_mr3`<br>Like this: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="lsi_mr3 mod 1" src="/images/esxi7-missing-percs/esxi7-missing-percs-05.png">
14. Save and quit vi
15. Next, edit the ids file:<br>`vi /vmfs/volumes/datastore1/tmp/usr/share/hwdata/default.pciids.d/lsi_mr3.ids`
16. Paste in the following in to the "Broadcom" block, conforming the tab formatting detailed at the top of the file:<br>
{% highlight shell %}
	0079
		1028 1f16  Dell PERC H700 Adapter
		1028 1f17  Dell PERC H700 Adapter Integrated
		1028 1f18  Dell PERC H700 Adapter Modular  
{% endhighlight %}
<img style="display: block; margin-left: auto; margin-right: auto;" alt="lsi_mr3 mod 2" src="/images/esxi7-missing-percs/esxi7-missing-percs-06.png">
17. Save and quit vi
18. Back in the tmp directory, compress one:<br>`tar -cf /vmfs/volumes/datastore1/FILE.tar *`
19. Change to root of datastore:<br>`cd /vmfs/volumes/datastore1/`
20. Compress two:<br>`vmtar -c FILE.tar -o output.vtar`<br>
(ignore the "not a valid exec file" error)
21. Compress three:<br>`gzip output.vtar`
22. Rename file:<br>`mv output.vtar.gz lsi_mr3.v00`
23. Clean up:<br>`rm FILE.tar`
24. Finally copy modified lsi_mr3.v00 back to boot bank:<br>`cp /vmfs/volumes/datastore1/lsi_mr3.v00 /bootbank/lsi_mr3.v00`
 
Phew!!!

Steps from above without comments (easier to read):
{% highlight shell %}
cp /tardisks/lsi_mr3.v00 /vmfs/volumes/datastore1/s.tar
cd /vmfs/volumes/datastore1/
vmtar -x s.tar -o output.tar
rm s.tar
mkdir tmp
mv output.tar tmp/output.tar
cd tmp
tar xf output.tar
rm output.tar
vi /vmfs/volumes/datastore1/tmp/etc/vmware/default.map.d/lsi_mr3.map
Paste in the following:
	regtype=native,bus=pci,id=10000079..............,driver=lsi_mr3
vi /vmfs/volumes/datastore1/tmp/usr/share/hwdata/default.pciids.d/lsi_mr3.ids
Paste in the following in to the "Broadcom" block:
	0079
		1028 1f16  Dell PERC H700 Adapter
		1028 1f17  Dell PERC H700 Adapter Integrated
		1028 1f18  Dell PERC H700 Adapter Modular  
tar -cf /vmfs/volumes/datastore1/FILE.tar *
cd /vmfs/volumes/datastore1/
vmtar -c FILE.tar -o output.vtar
(ignore the "not a valid exec file" error)
gzip output.vtar
mv output.vtar.gz lsi_mr3.v00
rm FILE.tar
cp /vmfs/volumes/datastore1/lsi_mr3.v00 /bootbank/lsi_mr3.v00
{% endhighlight %}


Finally.... Reboot!

## Testing
....aaaaand this is as far as I've got.  As I said above, whilst I've confirmed my ESXi 7.0 install with the modified lsi_mr3 driver still boots in as a VM, I've not yet been able to test this on my R710 server.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="straws" src="/images/esxi7-missing-percs/esxi7-missing-percs-07.png">

Am I confident that this will work? <br>Realistically,I'd give it about a 30% chance of working. After all, drivers change; functionality is added and removed all the time. 

Whilst we are all in isolation thanks to COVID-19, what's the harm in trying right? It'll either work or it won't!

Hopefully I can test soon.<br> 

In the meantime, if you fancy having a go at the above on your LSI 2108 based adapter, please be my guest. Just remember to use the PCIDs that match your hardware! :wink:

-Chris