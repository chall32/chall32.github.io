---
layout: post
title: "ESXi-Arm First Thoughts" 
excerpt: "Why I'm not running it.. yet"
tags: 
- Free
- Pro-Tip
- VMware
image:
  thumb: /esxi-arm-first-thoughts/esxi-arm-first-thoughts-00.png
comments: true
date: 2020-10-11T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="vSphere Logo" src="/images/esxi-arm-first-thoughts/esxi-arm-first-thoughts-01.png">
With the news this week of the release of the [ESXi-Arm fling](https://blogs.vmware.com/vsphere/2020/10/announcing-the-esxi-arm-fling.html), coupled with me already having a Raspberry Pi (RPi) version 4 with 4GB in my possession, you'd think that I'd be all over ESXi-Arm, wanting to immediately take it out for a test drive.

Well, no, not really...

Getting into the whole RPi game a little late, I've had my RPi since May this year.  In the time between my RPi purchase and the ESXi-Arm announcement earlier this week, ESXi has not been an option for my RPi.  

Therefore after doing all the usual stuff like:

- Installing [Raspbian](https://www.raspbian.org/)
- Turning RPi into a [retro gaming rig](https://retropie.org.uk/)
- Turning RPi into a [Media Player](https://kodi.wiki/view/Raspberry_Pi) with [Plex support](https://mediaexperience.com/raspberry-pi-xbmc-with-raspbmc/)

All of which is a super cool and all. Props to all those involved in all of the above projects, keep up the good work!

I wanted to try turning my RPi into more of a server platform and perhaps migrate some of the workload of my existing R710 ESXi host onto the RPi. With ESXi-Arm not being available back then and after weighing up the other options, I felt I was left with one option that interested me:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Docker!" src="/images/esxi-arm-first-thoughts/esxi-arm-first-thoughts-04.png">

Besides, if ESX-Arm *had* been available at the time, where's the fun in just installing that?

To me - as a virtualisation admin - docker offered something new that I wanted to learn more about: **containerisation**. 

Sure, I'd played with docker a number of times in labs before, but nothing "production ready".

Plus, what better way to learn than by doing? I had:

- A need:  To migrate some workloads off of ESXi 
- A target: Docker on RPi running on Ubuntu 20.04LTS Server
- An interest to learn about Docker

So after following some guides - there is little point in posting how to guides here when many, excellent guides already exist:

- Installing Ubuntu 20.04LTS Server on RPi: [Ubuntu Tutorials](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview)
- Installing and using Docker on Ubuntu 20.04 [Digital Ocean Tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)
- Running Portainer within Docker [Portainer.io Install Guide](https://www.portainer.io/installation/)

Today I have six containers running on my RPi so far:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="RPi Containers" src="/images/esxi-arm-first-thoughts/esxi-arm-first-thoughts-02.png">

With my RPi barely breaking a sweat:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="RPi top" src="/images/esxi-arm-first-thoughts/esxi-arm-first-thoughts-03.png">

With plans for more containers in the pipeline. 

## Conclusion and Wrap Up
So in conclusion; no I'm not running ESXi-Arm on my Raspberry Pi. That would have been the easy option.

Like millions of others, I'm using my Raspberry Pi for education. In my case containerisation education. Once I have the basics of containerisation down, I can then look at [Kubernetes](https://kubernetes.io/) and maybe [VMware Tanzu](https://tanzu.vmware.com/tanzu) one day.

Until then, I'm happy to continue to use my Raspberry Pi to add the containerisation [string to my bow](https://www.collinsdictionary.com/dictionary/english/a-string-to-ones-bow#:~:text=If%20someone%20has%20more%20than,many%20strings%20to%20my%20bow.&text=Collins!).

-Chris