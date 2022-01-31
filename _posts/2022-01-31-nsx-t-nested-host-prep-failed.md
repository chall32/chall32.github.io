---
layout: post
title: "NSX-T Nested ESXi Host Preparation Failed or Timed Out" 
excerpt: "Preparation H... H for what the Sam Hill?"
tags: 
- Pro-Tip
- VMware
- Deployment
- ESXi
- NSX-T
image:
  thumb: /nsx-t-nested-host-prep-failed/nsx-t-nested-host-prep-failed-00.jpg
comments: true
date: 2022-01-31T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="NSX-T Logo" src="/images/nsx-t-nested-host-prep-failed/nsx-t-nested-host-prep-failed-01.jpg">
I've just spent a large chunk of my day trying to complete and then troubleshoot something that should have been an easy task...

The task was to deploy a quick NSX-T into a vSphere 7.0 environment nested under vSphere 6.7.  One ESXi, one vCenter, one NSX-T manager. The same as I had done for my post [NSX-T 3.2: Micro-Segmentation Only Deployment - Manual Setup](https://polarclouds.co.uk/nsx-t-3-2-manual-microsegmentation/){:target="_blank"}.

What's more is that the lab deployed for that post was also nested - albeit nested under vSphere 7.0 environment and that lab deployed just fine, no issues at all!!

However I approached it (NSX-T wizard or no wizard), I could not get the host prepared beyond "Waiting for connection to Managers":

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Stuck!" src="/images/nsx-t-nested-host-prep-failed/nsx-t-nested-host-prep-failed-02.png"> 

The hard part - VIB installation - was all over and done. What's happening here?

After a lot of trial and error, a bit of Googling led me to a blog post [NSX-T Nested ESXi host preparation fails](https://vxsan.com/nsx-t-esxi-host-preparation-fails-errno-1-operation-not-permitted-it-is-not-safe-to-continue/){:target="_blank"} from Sjors Robroek. 

After checking, yep I had secure boot enabled:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Secure Boot" src="/images/nsx-t-nested-host-prep-failed/nsx-t-nested-host-prep-failed-03.png"> 

Power the host off, tweak it's firmware back to BIOS:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="BIOS Please" src="/images/nsx-t-nested-host-prep-failed/nsx-t-nested-host-prep-failed-04.png"> 

A manual clean up of the ESXi host following: [Quick Tip: NSX-T 3.0: Removing VIBs manually from ESXi host](https://patrik.kernstock.net/2020/07/quick-tip-nsx-t-3-0-removing-vibs-manually-from-esxi-host/){:target="_blank"} by Patrik Kernstock.

**Do read and take heed of Patriks warnings before running!** I had zero VMs and I already reinstalled my ESXi server once, so nothing to loose. :unamused:
{% highlight shell %}
nsxcli
del nsx
vsipioctl clearallfilters -Override
esxcli software vib remove -n=nsx-adf -n=nsx-context-mux -n=nsx-exporter -n=nsx-host -n=nsx-monitoring -n=nsx-netopa -n=nsx-opsagent -n=nsx-proxy -n=nsx-python-logging -n=nsx-python-utils -n=nsxcli -n=nsx-sfhc -n=nsx-platform-client -n=nsx-cfgagent -n=nsx-mpa -n=nsx-nestdb -n=nsx-python-gevent -n=nsx-python-greenlet -n=nsx-python-protobuf -n=nsx-vdpi -n=nsx-ids; esxcli software vib remove -n=nsx-esx-datapath --no-live-install; esxcli software vib remove -n=vsipfwlib -n=nsx-cpp-libs -n=nsx-proto2-libs -n=nsx-shared-libs
{% endhighlight %}

Reboot the host, try again and oh look, hey presto:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Host Prepared!" src="/images/nsx-t-nested-host-prep-failed/nsx-t-nested-host-prep-failed-05.png"> 

File this one under: 
- Differences between vSphere 6.7 and vSphere 7.0
- Update your host firmware BEFORE deploying NSX-T (if deploying to physical servers)
- Secure boot?  Secure pain in the ...!
- :musical_note: *U h8 UEFI, I h8 UEFI, lets got back to BIOS* :musical_note:

Joy. I'm off to find a darkened room.

-Chris