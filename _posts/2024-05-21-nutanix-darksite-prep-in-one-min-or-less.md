---
layout: post
title: "Nutanix Dark Site Preparation in One Minute or Less"
excerpt: "You get a dark site, you get a dark site and you do too"
tags: 
- Nutanix
- PowerShell
image:
  thumb: nutanix-darksite-prep-in-one-min-or-less/nutanix-darksite-prep-in-one-min-or-less-00.png
comments: true
date: 2024-05-21T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Nutanix Darksite" src="/images/nutanix-darksite-prep-in-one-min-or-less/nutanix-darksite-prep-in-one-min-or-less-00.png">
Whilst it is a requirement for a cluster based on Nutanix Community Edition to have internet access to enable cluster login, it is not a set requirement for Nutanix enterprise clusters to have internet access.

In such cases to allow for cluster patching, updates, application deployments and add-ons, these clients can leverage what is called a Dark Site server. This server typically takes the form of a web server located on the client network that the Nutanix infrastructure can reference via Life Cycle Manager (LCM).

The web server can be Linux or Windows based with the details of the the configuration required to deploy such a web server can be found in the [Preparing Your Cluster Using a Local Web Server](https://portal.nutanix.com/page/documents/details?targetId=Life-Cycle-Manager-Dark-Site-Guide-v2_7:Preparing%20Your%20Cluster%20Using%20a%20Local%20Web%20Server){:target="_blank"} section of the Life Cycle Manager Dark Site Guide documentation.

As you will see from the [Setting Up a Local Web Server (Windows)](https://portal.nutanix.com/page/documents/details?targetId=Life-Cycle-Manager-Dark-Site-Guide-v2_7:top-lcm-darksite-web-server-windows-t.html){:target="_blank"} section of the documentation, there is a moderate amount of configuration required to prepare a Windows Internet Information Services (IIS) server to become a dark site web server:

- Add Windows Firewall Rule for port 80 (http)
- Install IIS
- Create IIS Mime types
- Create release and builds folders
- Enable IIS Directory browsing
- Extract LCM Framework Bundle from supplied file

Sure, whilst configuring IIS is ~~super fun~~ (not), I can think of more exciting things to be doing with my time.  What we need here is a script to complete the end-to-end IIS configuration for us. 

It just so happens that I've created a script that will take a newly deployed Windows 2022 VM and apply all the configuration required.


Let's take a look at the New-Darksite.ps1 script:

{% highlight powershell %}
<# New-Darksite.ps1 
   - v0.1 - 13 Dec 2023 - Chris Hall - Initial release
   - v0.2 - 17 Jan 2024 - Chris Hall - Use local LCM Framework Bundle
   - v0.3 - 13 May 2024 - Chris Hall - Tweak prompt, add Firewall Rule, add LCM URL 
   
   Monitor IIS logs:  Get-Content C:\inetpub\logs\LogFiles\W3SVC1\u_exYYMMDD.log -Wait -Tail 10

.SYNOPSIS
   Create a Windows 2022 Server Based Nutanix Dark Site Web Server
.DESCRIPTION
   This script will prepare a freshly installed Windows 2022 server to become a Nutanix Life Cycle Manager (LCM) dark site web server 
.INPUTS
   Freshly installed Windows 2022 Server without IIS enabled (Tested against version 21H2)
   Browse to https://portal.nutanix.com/page/downloads?product=lcm and download LCM Framework Bundle
.OUTPUTS
   Windows 2022 based IIS web server configured ready to host Nutanix LCM updates for a dark site
   See https://portal.nutanix.com/page/documents/details?targetId=Life-Cycle-Manager-Dark-Site-Guide-v2_7:Fetching%20LCM%20Software%20Update%20Bundles%20Using%20a%20Web%20Server
   for bundle placement(s)
.NOTES
   Process taken From https://portal.nutanix.com/page/documents/details?targetId=Life-Cycle-Manager-Dark-Site-Guide-v2_7:top-lcm-darksite-web-server-windows-t.html
.FUNCTIONALITY
   The script will complete the following:
   - Add Windows Firewall Rule for port 80 http
   - Install IIS
   - Create IIS Mime types
   - Create release and builds folders
   - Enable IIS Directory browsing
   - Extract LCM Framework Bundle from supplied file
.DISCLAIMER
   See https://polarclouds.co.uk/pages/disclaimer/ 
.LICENSE
   See https://www.nutanix.dev/code_samples/#code_license
#>
# Reminder
Clear-Host
$Reminder = Read-Host "You did remember to download the LCM Framework Bundle from https://portal.nutanix.com/page/downloads?product=lcm ? - I'll ask for it later "
if ($Reminder -eq "N"){ Exit }

# Add Windows Firewall Rule 
New-NetFirewallRule -DisplayName 'Nutanix Dark Site Web Server' -Profile Any -Direction Inbound -Action Allow -Protocol TCP -LocalPort 80

# Install IIS
Install-WindowsFeature Web-Server -IncludeManagementTools
Start-Sleep 10

# Create IIS Mime types
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.sign',mimeType='text/plain']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.iso',mimeType='text/plain']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.xz',mimeType='text/plain']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.frm',mimeType='application/x-iso9660-image']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.ftd',mimeType='application/x-iso9660-image']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.lod',mimeType='application/x-iso9660-image']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.md',mimeType='application/x-iso9660-image']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.qcow2',mimeType='application/x-iso9660-image']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.std',mimeType='application/x-iso9660-image']"
& $Env:WinDir\system32\inetsrv\appcmd.exe set config /section:staticContent /+"[fileExtension='.vib',mimeType='application/x-iso9660-image']"

# Create release and builds folders
New-Item "C:\inetpub\wwwroot\release" -ItemType Directory -Force 
New-Item "C:\inetpub\wwwroot\release\builds" -ItemType Directory -Force 

# Enable IIS Directory browsing 
$siteName = "Default Web Site"
$site = Get-IISSite -Name $siteName
$apps = $site.Applications
ForEach ($app In $apps) {
    Set-WebConfigurationProperty -PSPath "MACHINE/WEBROOT/APPHOST/$site$($app.Path)" -Filter "system.webServer/directoryBrowse" -Name "enabled" -Value "True"
} 
 # Extract LCM Framework Bundle
Add-Type -AssemblyName System.Windows.Forms
Try {
    Write-Host "Downloaded LCM Framework Bundle Please"
    $LCMDSBBrowser = New-Object System.Windows.Forms.OpenFileDialog -Property @{ 
    InitialDirectory = $env:homedrive
    Filter = 'Gzip File (*.gz)|*.gz|All files (*.*)|*.*' }
    $null = $LCMDSBBrowser.ShowDialog()
    $LCMDSB = $($LCMDSBBrowser.FileName)
 }
Catch {
    Write-Host "Nope. Giving up!"
    Exit
}
Move-Item "$LCMDSB" c:\inetpub\lcm_dark_site_bundle.tar.gz -Force
cmd.exe /c 'tar xvfz c:\inetpub\lcm_dark_site_bundle.tar.gz --directory c:\inetpub\wwwroot\release'
Write-Host "`n`nDONE!`n`nDownload LCM bundles and extract to C:\inetpub\wwwroot\release\builds folder"
$IP = (Get-NetIPAddress -AddressFamily IPv4 | where IPAddress -notlike '127.0.*').IPAddress
Write-Host "`n`nConfigure Prism Element/Central with LCM Dark Site URL: http://$IP/release"
{% endhighlight %}

As detailed in the script, it has been fully tested on Windows Server 2022 (21H2), an evaluation version of which can be downloaded direct from Microsoft here:

[go.microsoft.com](https://go.microsoft.com/fwlink/p/?LinkID=2195280&clcid=0x409&culture=en-us&country=US){:target="_blank"}

Having said that, the script *should* work without issue on Windows Server 2019.

After running the New-Darksite.ps1, simply download and extract LCM bundles to `C:\inetpub\wwwroot\release\builds` folder.

That's it, a pid and effortless way to create a Nutanix dark site web server.

For other scripts, code samples, API reference and more, check out:

[<img style="display: block; margin-left: auto; margin-right: auto;" alt="Nutanix.dev" src="/images/nutanix-darksite-prep-in-one-min-or-less/nutanix-darksite-prep-in-one-min-or-less-01.png">](https://www.nutanix.dev){:target="_blank"}

Very quick post this week as I'm here:

[<img style="display: block; margin-left: auto; margin-right: auto;" alt=".next 2024" src="/images/nutanix-darksite-prep-in-one-min-or-less/nutanix-darksite-prep-in-one-min-or-less-02.png">](https://www.nutanix.com/next){:target="_blank"}

If you see me, don't hesitate to say hello!

-Chris