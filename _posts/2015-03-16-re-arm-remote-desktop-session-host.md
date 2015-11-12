---
layout: post
title: Re-Arm Remote Desktop Session Host
date: '2015-03-16T12:16:00.000Z'
tags:
- RDP
- Windows
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Modem Tweet" src="/images/Remote_desktop_connection_icon.png">
So here is a little fix for an unlicensed remote desktop session host I found...

{% include _toc.html %}

### Scenario

You have enabled remote desktop session host (also known as remote desktop terminal services mode) in trial mode on a Windows 2012 or Windows 2012R2 server some time ago and now you are receiving the error:  

>"The remote session was disconnected because there are no Remote Desktop Licence Servers available to provide a licence. Please contact the server administrator"

You may also notice Event ID: 1128 Source: TerminalServices-RemoteConnectionManager being logged in your system event log.  

### Cause

You are outside of your 120 day remote desktop session host evaluation period and / or the service has not been configured to register with a license server to install licenses.  A remote desktop licensing server is required for continuous normal operation.  

### Resolution 1

Install a remote desktop licensing server with the appropriate number of remote desktop session host licences and register your session host server with this.  

### Resolution 2

Re-arm your remote desktop session host evaluation to allow for another 120 days evaluation time. Here is how:  

1. Logon to your remote desktop session host server, open up regedit and navigate to

   `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\RCM\GracePeriod`

2. Right click GracePeriod key and select Permissions.  Grant Administrators full control as shown below: 

   ![](/images/gp.jpg)

3. Delete the `L$RTMTIMEBOMB` value leaving only the `(default)` value
4. Reboot your remote desktop session host server
5. Job done. You should have another 120 days evaluation time 

I understand that this resolution also works for Windows 2008, Windows 2008R2  As well as Windows 2012 and Windows 2012R2.  

-Chris
