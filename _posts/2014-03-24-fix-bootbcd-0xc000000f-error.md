---
layout: post
title: Fix Boot/BCD 0xc000000f Error
date: '2014-03-24T08:00:00.000Z'
tags:
- Windows
- Reminder
---
File this one under a post for another day / ah yes, I've seen that before, cant remember how I fixed it however... 

{% include _toc.html %}

![](/images/bcderror.jpg)

That's:

> File: \Boot\BCD  
> Status: 0xc000000f  
> Info: an error occurred while attempting to read the boot configuration data

Oh joy... OK, here is how to fix:  

### Step 0: Getting to the Recovery Console

1. Insert Windows DVD* and after selecting language and keyboard, select "Repair your computer"  
2. Wait for system recovery to run and fail  
3. Click "No" to apply any changes  
4. Cick "Next" to look for a recovery image  
5. Click "Cancel" on the cannot find system image dialogue  
6. Click "Cancel" to exit system image dialogue  
7. Click Command Prompt  

### Step 1: Ensure your system partition is marked as active

As a reminder - <span style="font-family: Courier New, Courier, monospace;">**this is a typed command**</span>  
And this is a comment.  

1. Boot into the recovery console as per step 0  
2. <span style="font-family: Courier New, Courier, monospace;">**diskpart**</span>  
3. <span style="font-family: Courier New, Courier, monospace;">**select disk 0**</span>  
4. <span style="font-family: Courier New, Courier, monospace;">**list partition**</span>  
5. Select the first primary partition. In the screenshot below, the partition to select is partition 2, so <span style="font-family: Courier New, Courier, monospace;">**select partition 2**</span>:  

   ![](/images/diskpart.jpg)

6. <span style="font-family: Courier New, Courier, monospace;">**detail partition**</span>  
7. Ensure that the partition is marked as Active: Yes  

   ![](/images/active.jpg)

8. If not, then <span style="font-family: Courier New, Courier, monospace;">**active**</span> to set the partition active  
9. <span style="font-family: Courier New, Courier, monospace;">**exit**</span> to exit diskpart  
10. <span style="font-family: Courier New, Courier, monospace;">**exit**</span> to exit recovery console
11. Restart to reboot  
12. Boot and follow step 0 to enter the recovery console again  

### Step 2: Repair Master Boot Record and Repair Boot Sector

1. Boot back into the recovery console, as per step 0, run the following commands
2. <span style="font-family: Courier New, Courier, monospace;">**bootrec /fixmbr**</span>  
3. <span style="font-family: Courier New, Courier, monospace;">**bootrec /fixboot**</span>  

### Step 3: Rebuild Boot files

You need to know where your Windows folder is mounted within the recovery console. Sometimes it is at C:\Windows, sometimes D:\Windows, sometimes somewhere else. If you have no idea, use the following to get you a list of drive letters currently in use:  

1. <span style="font-family: Courier New, Courier, monospace;">**diskpart**</span>  
2. <span style="font-family: Courier New, Courier, monospace;">**select disk 0**</span>  
3. <span style="font-family: Courier New, Courier, monospace;">**list volume**</span>  

Then it's just a matter of looking for Windows directories on each of those volumes.  

So to rebuild the boot files:  
<span style="font-family: Courier New, Courier, monospace;">**bcdboot C:\Windows /s C:**</span>  
Reboot and you should be done.  

*If you can't find your Windows DVD, have a look  

[Here](http://www.w7forums.com/threads/official-windows-7-sp1-iso-image-downloads.12325/) for Windows 7 DVDs (Release versions)  
[Here](http://www.microsoft.com/en-gb/download/details.aspx?id=11093) for Windows 2008R2 DVD (Evaluation version)  
[Here](http://technet.microsoft.com/en-US/evalcenter/hh699156.aspx) for Windows 8.x DVD (Evaluation version)  
[Here](http://msdn.microsoft.com/en-gb/evalcenter/hh708764.aspx) for Windows 2012 DVD (Evaluation version)  

-Chris
