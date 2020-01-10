---
layout: post
title: "Running an OpenVPN Server on pfSense. Part 2: VPN Client"
excerpt: The client side of the equation  
tags: 
- Security
- VPN
- Pro-Tip
- Free
image:
  thumb: pfsense-vpn-client/pfsense-vpn-client-00.png
comments: true
date: 2020-01-10T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="pfSense + OpenVPN" src="/images/pfsense-vpn-client/pfsense-vpn-client-01.png">
Last time we looked at deploying an OpenVPN server on pfSense.  This time, let's look at setting up our clients to access the VPN server.

If you haven't had time to check out how we configured our OpenVPN server, feel free to [take a look](https://polarclouds.co.uk/pfsense-vpn-server/).

Luckily enough once again, this is where the pfSense team have done the heavy lifting for us making our life so much easier!  Lets get started.

{% include _toc.html %}
## OpenVPN Client Export Utility
The OpenVPN Client Export utility is an add-on package for pfSense.  Once installed, it can automatically create a Windows OpenVPN client installer to download, or it can generate configuration files for Android, Apple iOS, create Viscosity bundles for MAC OSX and others. Lets look at installing and using this add-on.

## Installing OpenVPN Client Export Utility Package
Log onto your pfSense server [created last time](https://polarclouds.co.uk/pfsense-vpn-server/) and navigate to **System / Package Manager / Available Packages** and search for "openvpn":

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Find Package" src="/images/pfsense-vpn-client/pfsense-vpn-client-02.png">

Once found, click **Install** and **Confirm** to install the package and allow to complete:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OpenVPN Client Package Install" src="/images/pfsense-vpn-client/pfsense-vpn-client-03.png">

## OpenVPN Client Export Utility Configuration
Navigate to **VPN / OpenVPN / Client Export**
Complete the following:

- **Remote Access Server** - Should auto select the OpenVPN Server already installed
- **Host Name Resolution** - Set to "Other"
- **Host Name** - Enter either your [Public IP Address](https://www.google.co.uk/search?hl=en&q=whats+my+ip+address) or [hostname](https://www.hostip.info) here.

**Note:** If you have a non static public IP address, IE one that changes every time you reboot your router, use a you'll need to use a [DynamicDNS service]( https://www.noip.com/blog/2014/07/11/dynamic-dns-can-use-2/) and [configure it appropriately]( https://www.noip.com/support/knowledgebase/how-to-configure-ddns-in-router/)

- **Verify Server CN** - Automatic - Use verify-x509-name (OpenVPN 2.3+) where possible
- **Block Outside DNS** - Ticked
- **Legacy Client** - Unticked
- **Use Random Local Port** - Unticked
- **PKCS#11 Certificate Storage** - Unticked
- **Microsoft Certificate Storage** - Unticked
- **Password Protect Certificate** - Unticked
- **Use A Proxy** - Unticked
- **Additional configuration options** - Leave blank

Click **Save as Default** to save the above settings

## Using OpenVPN Client Export Utility
Now the fun part!
### Android OpenVPN Client Installation
The recommended client for Android is [OpenVPN for Android](https://play.google.com/store/apps/details?id=de.blinkt.openvpn) <br>
Install the recommended client, find the OpenVPN user and download the Android inline configuration: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Android Config Download" src="/images/pfsense-vpn-client/pfsense-vpn-client-04.png">

Copy the downloaded configuration to the Android phone, import using OpenVPN Client **(+)** option and name the connection. Tap the connection name and test:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OpenVPN Android" src="/images/pfsense-vpn-client/pfsense-vpn-client-06.png">

### Windows OpenVPN Client Installation
Simply find the OpenVPN user and the appropriate installer for their version of windows:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Windows installers" src="/images/pfsense-vpn-client/pfsense-vpn-client-05.png">

Install and test:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OpenVPN Windows" src="/images/pfsense-vpn-client/pfsense-vpn-client-07.png">

### Apple iOS OpenVPN Client Installation
The recommended client for iOS is [OpenVPN Connect](https://apps.apple.com/us/app/openvpn-connect/id590379981) <br>
Install the recommended client, find the OpenVPN user and download the inline configuration: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="iOS Config Download" src="/images/pfsense-vpn-client/pfsense-vpn-client-08.png">

Attach the configuration to an email and open the email on the iOS device. <br>
Tap the attachment and open it in the OpenVPN Connect app. Click **Add** to add the profile, rename if needed, add a username and click **Add**. 

Allow the app to add VPN connections in iOS settings and finally hit the slider to test:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OpenVPN iOS" src="/images/pfsense-vpn-client/pfsense-vpn-client-09.png">

### Mac OS X OpenVPN Client Installation
Yep it's available:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="OpenVPN OS X" src="/images/pfsense-vpn-client/pfsense-vpn-client-10.png">

Unfortunately at this point, I don't have any experience with installing and testing. <br>
[Perhaps one day](https://hackintosh.com/) :smirk:

## Conclusion
And there we have it!<br>

An OpenVPN server set up and OpenVPN clients to match. How much did it cost?  Nothing, just a bit of time and patience. <br>

Security doesn't need to be expensive. :sunglasses:

-Chris