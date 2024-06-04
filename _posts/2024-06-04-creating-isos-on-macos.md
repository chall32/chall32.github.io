---
layout: post
title: "Creating ISOs on MacOS"
excerpt: "Using Built-in Tools, No apps required"
tags: 
- Pro-Tip
- MacOS
image:
  thumb: create-isos-on-macos/create-isos-on-macos-01.png
comments: true
date: 2024-06-04T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="MacOS ISO" src="/images/create-isos-on-macos/create-isos-on-macos-01.png">
Simple little post as I can never remember how to create CD or DVD ISO files on MacOS. 

Yes, I'm sure there are apps in the App Store or available online to allow me to create ISOs, but sometimes you (I) really, really don't want to pay for something that I'm only going to use once in a blue moon.

Surely there must be a method to use MacOS native apps rather than paying for something? 

**The answer is yes, there is a way to create ISO files on MacOS.**

So a quick post to save me from having to using a search engine each time to find that one post. It’s a two step procedure, first you use Disk Utility to create a CDR image, then you convert that image to an ISO using the hdiutil utility. Props go to [Pete Long](https://www.petenetlive.com/KB/Article/0001554){:target="_blank"} for the method reposted here.

{% include _toc.html %}
### 1. Create CDR - Disk Utility

Launch **Disk Utility > File > Image from Folder > Browse** to and select the folder containing your files to be written to a CD/DVD image:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create CDR 1" src="/images/create-isos-on-macos/create-isos-on-macos-02.png">

Choose where to save the image, set Encryption to **None** and **Image Format** to DVD/CD master:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create CDR 2" src="/images/create-isos-on-macos/create-isos-on-macos-03.png">

### 2. Convert CDR to ISO - hdiutil
Finally, we need to use the command line utility `hdiutil` to convert the .cdr file to an .iso file:

{% highlight text %}
hdiutil makehybrid -iso -joliet -o -FILENAME.iso FILENAME.cdr
{% endhighlight %}

The hdiutil parameters used are as below:

- makehybrid : generate cross-platform hybrid images
- -iso : Generate an ISO9660 filesystem
- -joliet : Generate Joliet extensions to ISO9660
- -o : Output filename
- Source filename

Which when run:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create ISO 1" src="/images/create-isos-on-macos/create-isos-on-macos-04.png">

Job done.

The resulting images can be used to boot virtual machines via your favourite virtualisation software or physical machines via USB using [Ventoy](https://www.ventoy.net){:target="_blank"}  or over the network using [iVentoy](https://www.iventoy.com){:target="_blank"}.

-Chris