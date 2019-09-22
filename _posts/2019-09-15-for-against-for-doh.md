---
layout: post
title: Why I was For, Against, then For Browser DNS over HTTPS
excerpt: "Change your mind much? DoH..."
tags:
- Adverts
- Pro-Tip
- Free
image:
  thumb: for-against-for-doh/for-against-for-doh-00.jpg
comments: true
date: 2019-09-15T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;"  alt="DoH Nut" src="/images/for-against-for-doh/for-against-for-doh-01.jpg">
Last time we looked at enabling Pi-hole to keep internet bourne garbage (adverts, ransomware, crypo-miners, internet tracking, malware, etc) off of all devices connected to our network.  As a privacy bonus we also enabled DNS over HTTPS (DoH) for internet bound DNS traffic from/to our network. 

Check out [Pi-hole plus DNS over HTTPS](https://polarclouds.co.uk/pi-hole-https-dns/) for the full how to.

Since then, DoH has exploded.  For example:

- Mozilla Firefox to begin slow rollout of DNS-over-HTTPS by default at the end of the month - [The Register](https://www.theregister.co.uk/2019/09/09/mozilla_firefox_dns/)
- Google experiments with DNS-over-HTTPS in Chrome - [Naked Security](https://nakedsecurity.sophos.com/2019/09/12/google-experiments-with-dns-over-http-in-chrome/) 
- Experimenting with same-provider DNS-over-HTTPS upgrade - [Chromium Blog](https://blog.chromium.org/2019/09/experimenting-with-same-provider-dns.html) 
- Rolling in DoH: Chrome 78 to experiment with DNS-over-HTTPS – hot on the heels of Firefox - [The Register](https://www.theregister.co.uk/2019/09/10/chrome_78_dnsoverhttps/)
- Firefox DNS-over-HTTPS - [Mozilla Support](https://support.mozilla.org/en-US/kb/firefox-dns-over-https)

Plus [many others](https://news.google.com/search?q=dns+over+https)

## For
My first reactions were: <br>
Excellent! <br>
More privacy for all!  <br>
Brilliant! <br>
:thumbsup: :sunglasses: :thumbsup: Happy days!

## Against
I then thought some more...<br>
My browser is now using it's own DNS server... separate to my internet garbage blocking Pi-hole DNS server (or any DNS based blocking service)...<br>
That means all previously blocked internet garbage will return! <br>

:thumbsdown: :cold_sweat: :thumbsdown: Oh "balls", that's not good!

## For ...Again!
After a little bit more reading, it appears that this concern was shared by others too.  

Luckily Mozilla are [ahead of the game](https://support.mozilla.org/en-US/kb/configuring-networks-disable-dns-over-https).
Essentially, their idea is to build into Firefox a check where by the browser will query DNS for a certain "canary" domain. The result returned from DNS will govern whether the browser switches to from standard (non-DoH) DNS to DoH or not.

The logic is as follows. First the browser makes the query to standard (non https) DNS for [use-application-dns.net](https://use-application-dns.net/). Standard DNS will then return one of the following:

|         Standard DNS Return         |          Browser Action              |
|:-----------------------------------:|:------------------------------------:|
| A or AAAA records                   | Enable DNS over HTTPS Functionality  |
| (valid IP addresses)                | (bypassing standard DNS)             |
|:-----------------------------------:|:------------------------------------:|
| NXDOMAIN or SERVFAIL                | Disable DNS over HTTPS Functionality |
| (unable to find valid IP addresses) | (continue to use standard DNS)       |
|:-----------------------------------:|:------------------------------------:|
|                                     |                                      |
{: rules="groups"}
Therefore to continue using Pi-hole to block internet garbage, pi-hole *must* return NXDOMAIN or SERVFAIL when queried for [use-application-dns.net](https://use-application-dns.net/). 

What's more is that those top, top Pi-hole developers have already merged a fix to [make this happen](https://github.com/pi-hole/pi-hole/pull/2915). Further discussion on this change is available on a pi-hole [discourse thread](https://discourse.pi-hole.net/t/support-for-returning-nxdomain-for-use-application-dns-net-to-disable-firefox-doh/23243/7).

Google's suggested implementation is [way more complex](https://docs.google.com/document/d/15Ss0OaJeb-T3g2RMwgikHvsC0CPKd-MLeGeetv1wYY4/edit?usp=sharing). I confess to not reading the whole document (it's 22 pages long!), however the issue is being thought about at least.  As with all suggestions, Google's implementation may change in the future...

Microsoft Edge supporting DoH?  The silence is deafening :no_mouth:

Fun times!

-Chris
