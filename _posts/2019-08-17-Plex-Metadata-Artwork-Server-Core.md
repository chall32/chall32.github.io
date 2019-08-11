---
layout: post
title: "Fixed: Plex Missing Metadata and Artwork on Windows Server Core"
excerpt: Deployed Plex on Windows Server Core? Missing metadata and artwork? Here's the fix...
tags:
- Windows
- Pro-Tip
- Free
image:
  thumb: Plex-Metadata-Artwork-Server-Core/Plex-Thumb.png
comments: true
date: 2019-08-17T09:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Plex Logo" src="/images/Plex-Metadata-Artwork-Server-Core/Plex-Thumb.png">
Here’s how I fixed Plex TV metadata and artwork not downloading when running Plex server on Windows Server 2016 / 2019 Core. 

This fix has the advantages of:

1. Installing just the two required root CA certificates
2. The DigiCert certificate has a long validity time (12 years at time of posting), so will not need replacing anytime soon

The AddTrust certificate expires on 30 May 2020. If you are reading this post after that date, you may want to double check the certificate you downloading the steps below.  Chances are Cloudflare will refresh the certificate in plenty of time anyway, so the links below *should* still work with the refreshed certificate. 

Onto getting these installed then.

Download the "DigiCert Global Root CA" certificate from [DigiCert Trusted Root Authority Certificates](https://www.digicert.com/digicert-root-certificates.htm) You will need the cert that is valid until 10 November 2031 and has a thumbprint ending in 5436. <br>[Direct link to DigiCertGlobalRootCA.crt](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt)

Next, download the "AddTrust External CA Root" certificate from [Cloudflare SSL cipher, browser, and protocol support](https://support.cloudflare.com/hc/en-us/articles/203041594) You will need the cert that has a serial number of 1 and has a SHA-1 Fingerprint ending in 1868. <br>[Direct link to 1.crt](https://crt.sh/?d=1)

Copy both certificates to your Windows Core install, say "C:\Temp"

Logon to core server and launch PowerShell. Run the command `Set-Location Cert:\LocalMachine\CA`

Import the DigiCert certificate: `Import-Certificate C:\Temp\DigiCertGlobalRootCA.crt`

Import the AddTrust certificate: `Import-Certificate C:\Temp\1.crt`

Finally, enter `dir` and confirm that you see the following listed:

> A8985D3A65E5E5C4B2D7D66D40C6DD2FB19C5436 CN=DigiCert Global Root CA, OU=www.digicert.com, O=DigiCert Inc, C=US
> 02FAF3E291435468607857694DF5E45B68851868 CN=AddTrust External CA Root, OU=AddTrust External TTP Network, O=AddTrust…

Refresh all metadata on your Plex TV libraries, hey presto - metadata and artwork.

-Chris