---
layout: post
title: Resetting Lost SRM Passwords
excerpt: Resetting VMware Site Recovery Manager root, admin and database passwords
tags:
- VMware
- Pro-Tip
image:
  thumb: reset-lost-srm-passwords/reset-srm-passwd-thumb.png
comments: true
date: 2019-08-10T16:20:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Lost password, send help!" src="/images/reset-lost-srm-passwords/reset-srm-passwd-00.jpg">
<span class="image-credit" style="float: right; margin: 0px 0px 10px 10px;">Photo: <a href="https://unsplash.com/@shttefan?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">SHTTEFAN</a></span>

Lets face it, we have all been there.  We *thought* the passwords were securely saved in the correct place.  However, when needed those carefully stowed passwords, for whatever reason they are not there! A couple of password safe restores later, it's looking like we are going to have to reset...

This happened to me with three Linux appliance installations of VMware Site Recovery Manager (SRM).

What's worse is that each SRM install has three (!!) passwords set at install time:

* root
* Admin
* Database

Well friends, here is how to reset all three passwords.  Whilst the following has been extensively tested on SRM v8.2 (latest version at time of post), your mileage may differ in later versions. 
{% include _toc.html %}

## root Password Reset

As the appliance runs Photon OS, the process to reset the appliance root password roughly follows that of resetting Linux root passwords in general.

Reboot the appliance via **vCenter - Actions - Power - Restart Guest OS** <br>
At the Photon OS splash screen quickly hit `e` to edit the grub boot menu. 

You will be met with a screen that resembles the following:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="SRM Appliance default grub menu" src="/images/reset-lost-srm-passwords/reset-srm-passwd-01.png">

Move the cursor to the end of the line that begins with `linux /$photon_linux` <br>
Enter the following text at the end of the linux line `rw init=/bin/bash` so that it resembles the following:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="SRM Appliance modified grub menu" src="/images/reset-lost-srm-passwords/reset-srm-passwd-02.png">

Hit `F10` to boot the SRM appliance using the modified grub entry.  
The appliance will boot to a command prompt:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="SRM Appliance booted to command prompt" src="/images/reset-lost-srm-passwords/reset-srm-passwd-03.png">

Now we simply need to change the root password using the `passwd` command as shown below:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="SRM Appliance root password changed" src="/images/reset-lost-srm-passwords/reset-srm-passwd-04.png">

Finally, unmount the file system using the command `umount /` and reboot using the command `reboot -f`:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="SRM Appliance unmount and reboot" src="/images/reset-lost-srm-passwords/reset-srm-passwd-05.png">

Allow the appliance to boot normally. Test the newly reset password by logging in using the root account via the VM's console.

## Admin Password Reset

Compared to resetting the root password as shown above, resetting the SRM admin password is simple.<br>
Open the SRM appliance VM console and login using the root account.

At the prompt enter `passwd admin`<br>
You will then be prompted to enter a new password for the admin account:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="SRM Appliance reset admin password" src="/images/reset-lost-srm-passwords/reset-srm-passwd-06.png">

Test access via the SRM appliance administration website `https://<SRM Appliance IP Address>:5480`:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="SRM Appliance login" src="/images/reset-lost-srm-passwords/reset-srm-passwd-07.png">

## Database Password Reset

Finally, lets reset the SRM database password.  Of the three SRM passwords set at install time, resetting the database password is the simplest.

Open the SRM appliance VM console and login using the root account.

Next, enter the following command: `cat /opt/vmware/srm/conf/db:srmdb`

This will display the currently set database password in clear text:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Recovered SRM database password" src="/images/reset-lost-srm-passwords/reset-srm-passwd-08.png">

Login via the SRM appliance administration website `https://<SRM Appliance IP Address>:5480`, select **Access - Embedded database password - Change**, enter recovered and new passwords:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Change SRM database password" src="/images/reset-lost-srm-passwords/reset-srm-passwd-09.png">

## Wrap Up
In this article we covered how to reset the VMware Site Recovery (SRM) Linux appliance root, admin and database passwords. All simple enough using standard Linux commands and knowing where to look to find PostgreSQL database passwords when you need them. 

Any comments, questions, concerns feel free to post them in the comments below.

Because I'll know someone will ask: "POTTY - that's a strange hostname... Why?"<br>
The simple answer is that I use the [SpongeBob SquarePants](https://en.wikipedia.org/wiki/SpongeBob_SquarePants) naming standard in my testlab.  You all know [Potty the Parrot](https://nickelodeon.fandom.com/wiki/Potty_the_Parrot) right?

That's right, this guy!
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Potty the Parrot" src="/images/reset-lost-srm-passwords/Potty_the_Parrot.png">

Finally, remember to save your new passwords!!!

-Chris