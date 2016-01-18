---
layout: post
title: Root Free Android App Data Backup and Restore 
excerpt: Safeguard your saved games or app data. Move app data between devices. No root? No problem!  
tags:
- Deployment
- Windows
image:
  thumb: android-app-data-backup-restore/andbnr00.png
comments: true
date: 2016-01-18T18:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Android Backup and Restore" src="/images/android-app-data-backup-restore/andbnr00.png">
Probably a bit late for the Xmas/new year rush of new Android kit now... hey ho, I was going to write this post over the Xmas break... but there you go, better late than never.

So here is a great tip for backing up and restoring Android application data (including saved games!) or even copying those saved games to a new device. This method works for pretty much any Android app that comes as an install from the Google Play store.  Nothing is more annoying that getting to level 9999 on Candy Crush (yeah I know exaggeration, but you get my drift) and then have to start again when you get a new device or your device suffers some other fate and all is lost...    

The secret here is we are going to use the [Android Debug Bridge](http://developer.android.com/tools/help/adb.html) to do the hard work for us.  Whats more, we can complete all of this without having to [root](https://en.wikipedia.org/wiki/Rooting_(Android_OS)) our devices!! 

{% include _toc.html %}

### Setting Up Android Debug Bridge (ADB)
There are loads and loads of guides on the internet to help you set up ADB. I highly recommend using the [Minimal ADB and Fastboot Tool](http://forum.xda-developers.com/showthread.php?t=2317790) to save on downloading the whole Android emulator stack.

[This guide](http://lifehacker.com/the-easiest-way-to-install-androids-adb-and-fastboot-to-1586992378) to setting up ADB covers Windows, OS X and Linux.

Personally, I use my Linux Mint laptop to do all things Android / ADB using the [android-tools-adb](http://community.linuxmint.com/software/view/android-tools-adb) package. Android under Windows is a pain what with device drivers and all that faff. Linux is nice, just plug in you device and away you go.

### Enable USB Debugging On Your Device
Again, have a look at [this guide](http://adbdriver.com/documentation/how-to-enable-usb-debugging-mode-on-android-devices.html) to enabling USB debuging on your device. Simple.

### Choosing What to Back Up
This should be the easy part!  You know what games / apps you want to back up right? OK, for the sake of an example lets use [Block Puzzle](https://play.google.com/store/apps/details?id=biz.mtoy.blockpuzzle.revolution) as an example app for the backup and restore process.

I need to find the app's package name as it would be installed on the device.  The simplest way I've found to find an app's package name is look at the app's entry in google play.  Specifically the URL.  

Using our example: [https://play.google.com/store/apps/details?id=biz.mtoy.blockpuzzle.revolution](https://play.google.com/store/apps/details?id=biz.mtoy.blockpuzzle.revolution). From this I can tell that the app's package name is "biz.mtoy.blockpuzzle.revolution". Make a note of the package name.

The reason I chose block puzzle is that yep, I have just 1 or 360 plus completed levels on my Nexus 5 so far: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB01" src="/images/android-app-data-backup-restore/andbnr01.png">

### Backing Up App Data
Right, so you:

* have adb installed.  
* have enabled USB debugging on your device
* know the package names of the apps with data that what you want to backup

Lets double check that we have visibility of the device via ADB using the command:

{% highlight text %}
adb devices
{% endhighlight %}
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB02" src="/images/android-app-data-backup-restore/andbnr02.png">

You should see a device listed as shown above.  Your device ID will be different to mine.  

On to the backup.  The adb data backup command takes this form:

{% highlight text %}
adb backup -f <Backup-File> <Package-Name>
{% endhighlight %}

So plumbing block puzzle into this command:

{% highlight text %}
adb backup -f biz.mtoy.blockpuzzle.revolution.ab biz.mtoy.blockpuzzle.revolution
{% endhighlight %}

*biz.mtoy.blockpuzzle.revolution.ab* is the name of my data back up file
*biz.mtoy.blockpuzzle.revolution*  is the name of app whos data we are backing up 

I like to name my data backup files to match the package name as that make sense to me. You can call your data backup files whatever you like!

Like this:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB03" src="/images/android-app-data-backup-restore/andbnr03.png">

At the device end:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB04" src="/images/android-app-data-backup-restore/andbnr04.png">

Just hit "BACKUP MY DATA" and give it a moment.  The size of backup data varies from app to app. Some mamage a few kB.  The biggest backup I've seen is around the 50MB mark!

Double checking backup file size on my laptop, it looks like we have application data from the device!
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB05" src="/images/android-app-data-backup-restore/andbnr05.png">
 
Repeat the command for any other application data you would like to backup.  

### Restoring App Data
First off, install the application as normal.  For the demo restore here, I'm going to restore my saved games from my Nexus 5 phone to another device, my Nexus 7 (2012) tablet.  

So install the app:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB06" src="/images/android-app-data-backup-restore/andbnr06.png">

Oh no's! No saved games (as expected):
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB07" src="/images/android-app-data-backup-restore/andbnr07.png">

No fear.  The ADB data restore command takes this form:

{% highlight text %}
adb restore <Backup-File>
{% endhighlight %}

So plumbing block puzzle into this command:

{% highlight text %}
adb restore biz.mtoy.blockpuzzle.revolution.ab
{% endhighlight %}

Like this:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB08" src="/images/android-app-data-backup-restore/andbnr08.png">

At the tablet end:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB09" src="/images/android-app-data-backup-restore/andbnr09.png">

Hit "RESTORE MY DATA" and give it a moment. 

Lets launch block puzzle again and check.  

Winner, winner:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="ADB10" src="/images/android-app-data-backup-restore/andbnr10.png">

Right I'm off to play block puzzle on my tablet... :o)

Till the next time.

-Chris
