---
layout: post
title: "Update Installation Failed: vCenter is Non-Operational" 
excerpt: "Ouch, That Sounds Bad"
tags: 
- Free
- Pro-Tip
- VMware
image:
  thumb: vcenter-is-non-operational/vcenter-is-non-operational-00.png
comments: true
date: 2020-05-06T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Old vSphere Logo" src="/images/vcenter-is-non-operational/vcenter-is-non-operational-00.png">
Patching lab and production vCenters this fault often pops up and every time it does it does give me the one of those terrible heart stopping moments. 

You'd think with an error this bad, it would be the end of your vCenter server, time to start looking round for some vCenter installation files right?

Wrong! (phew)

Here is a screenshot of the error so you can enjoy the terror too:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OMG!" src="/images/vcenter-is-non-operational/vcenter-is-non-operational-01.png">

A closer view for your delectation:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OMG! Close Up" src="/images/vcenter-is-non-operational/vcenter-is-non-operational-02.png">

> Update installation failed, vCenter is non-operational

Don't panic! Simply carry on and patch vCenter using the workaround below. No rebuild necessary. :sunglasses::thumbsup: 

## The Workaround
Here's how I update vCenter.  In my experience this method allows for a much smoother patch experience.

First step: BACKUP!! You do backup your vCenter don't you? ([HERE you go](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vcenter.install.doc/GUID-3EAED005-B0A3-40CF-B40D-85AD247D7EA4.html)) :wink:

Next, check for free disk space on the vCenter server.  SSH to vCeneter, open shell and issue the `df -h` command:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="vC df -h" src="/images/vcenter-is-non-operational/vcenter-is-non-operational-03.png">

From there, exit shell and enter back into the appliance shell (see [VMware KB2100508](https://kb.vmware.com/s/article/2100508) for vCenter shell details and toggling between them).

Mount the vCenter patch ISO file via a VMRC session to the vCenter VM initiated from the ESXi server currently running vCenter server.

To stage the packages in the update ISO:<br>
{% highlight shell %}
software-packages stage --iso
{% endhighlight %}
Accept the licence agreement. To list the staged content:<br>
{% highlight shell %}
software-packages list --staged
{% endhighlight %}
Finally, kick off the patch install: 
{% highlight shell %}
software-packages install --staged
{% endhighlight %}

All of which looks like this in practice:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Patching vC" src="/images/vcenter-is-non-operational/vcenter-is-non-operational-04.png">

Once done, exit SSH session and reboot vCenter via the console:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Reboot vC" src="/images/vcenter-is-non-operational/vcenter-is-non-operational-05.png">

Unmount the ISO and it's job done.  Much easier on the heart! :sparkling_heart::sunglasses::thumbsup:

-Chris