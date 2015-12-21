---
layout: post
title: UCS Blade Discovery Failed
tags:
- Cisco
image:
  thumb: ucs-blade-discovery-failed/L-F.jpg
comments: true
date: '2013-07-03T17:43:00.001+01:00'
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Lost n Found" src="/images/ucs-blade-discovery-failed/L-F.jpg">
A simple job then; lift and shift some Cisco UCS blades from a legacy site to into the Datacentre to help with capacity for consolidation in the Datacentre.

Unfortunately a simple job turned into a bit of a nightmare with the destination UCS deciding not to play nicely with the recycled blades. 
Don't get me wrong here folks, Cisco Unified Computing System is a cool piece of kit that is challenging the way we look at hardware nowadays.  It is however not without it's [foibles](http://www.thefreedictionary.com/foibles) of which this is just one.

Thanks go to [@brettchannon](https://twitter.com/brettchannon) and the guys at [@VCE](https://twitter.com/VCE) for helping with the solution to this issue.

{% include _toc.html %}

### Symtoms
When you install a Cisco UCS blade that is has 1.x firmware installed into a chassis that is running a 2.x firmware, the following error can be seen:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="error" src="/images/ucs-blade-discovery-failed/Fault2.jpg">

Code: F1000034
Cause: fsm-failed
Description: [FSM:FAILED] Blade Discovery (FSM:sam:dme:ComputeBladeDiscover)

A re-acknowledge, power cycle, reseat will not allow the blade to be properly discovered.  Any firmware upgrades (other than a CIMC firmware upgrade) will remain in a "Scheduled" status.

### Cause
USB Legacy mode is set to disabled within the BIOS settings.

### Resolution
Complete the following resolution on each blade affected:

1. Open the KVM console of the affected blade (Equipment Tab > Chassis > Chassis containing affected blade > Servers > Affected Server > KVM Console):
<img style="display: block; margin-left: auto; margin-right: auto;" alt="reset" src="/images/ucs-blade-discovery-failed/Reset.JPG">

2. Hit Reset and OK the following warning:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="reset OK 1" src="/images/ucs-blade-discovery-failed/ResetOK1.JPG">

3. Choose Power Cycle and OK the following dialogue:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="reset OK 2" src="/images/ucs-blade-discovery-failed/resetOK2.JPG">

4. Hit F2 when prompted to enter the blade's BIOS setup:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="F2" src="/images/ucs-blade-discovery-failed/F2.JPG">

5. Once in the BIOS setup hit right arrow key to get to Advanced and down arrow to USB Configuration:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="M200 bios" src="/images/ucs-blade-discovery-failed/m200bios2.JPG">

6. Hit return to open USB configuration and hit down arrow and return to open Legacy USB Support option:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="USB Disabled" src="/images/ucs-blade-discovery-failed/USBDisabled.JPG">

7. Set Legacy USB Support to Enabled:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Enable USB" src="/images/ucs-blade-discovery-failed/EnableUSB.JPG">

8. Hit Esc and right arrow to select Exit tab and hit return to Save Changes and Exit:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="M200bios 4" src="/images/ucs-blade-discovery-failed/m200bios4.JPG">

9.  Close the KVM console and allow UCS to rediscover server. If you cannot wait, select Recover Server > Re-acknowledge > OK to force the UCS to rediscover the blade.

### More Information

I would love to know more about this error and how the USB mode setting within a blade can cause UCS to give up on a that blade altogether.  

Seems like a crazy simple fix to what - on the face of it - seems a pretty catastrophic error message. All in all we had this issue on 12+ blades and the USB legacy mode fix work on all of them.

Godda love UCS.....!

-Chris
