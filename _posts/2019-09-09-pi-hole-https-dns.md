---
layout: post
title: "Pi-hole plus DNS over HTTPS"
excerpt: "Adverts, malware, tracking... stopped. Performance and privacy... increased!"
tags:
- Adverts
- Pro-Tip
- Free
image:
  thumb: pi-hole-https-dns/pi-hole-dns-https-00.png
comments: true
date: 2019-09-09T18:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="pi hole" src="/images/pi-hole-https-dns/pi-hole-dns-https-01.png">
<span class="image-credit" style="float: right; margin: 0px 0px 0px 10px;">Photo: <a href="https://pi-hole.net/">pi-hole.net</a></span>
Quick primer:

- DNS: Domain Name System: The system by which domain names such as polarclouds.co.uk are converted to Internet Protocol (IP) addresses such as 104.31.82.123
- HTTPS: Hypertext Transfer Protocol Secure: Used for secure communication over a computer network, such as the Internet
- Pi: Raspberry Pi (shortened to Pi in this instance): A low cost, credit-card sized computer: [See this PolarClouds post from 2011!](https://polarclouds.co.uk/raspberry-pi-16-linux-pc/) Now available in it's [fourth iteration](https://www.raspberrypi.org/)
- Hole: An empty space in an object, usually with an opening to the object's surface, or an opening that goes completely through an object. [Source](https://dictionary.cambridge.org/dictionary/english/hole).

**UPDATE:** See [Why I was For, Against, then For Browser DNS over HTTPS](https://polarclouds.co.uk/for-against-for-doh/)

{% include _toc.html %}
## Objectives
It's no secret, I'm not a fan of adverts.  Over the years I've posted about my disdain for adverts [several times](https://polarclouds.co.uk/pages/categories#Adverts). I think this sums up my feelings nicely:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="The joy of not being sold anything" src="/images/pi-hole-https-dns/pi-hole-dns-https-02.png">

Luckily for us there exists a free product that can extend this advert free joy to **EVERY DEVICE ON YOUR NETWORK** (computers, phones, tablets, you name it).

Also, let's block other internet borne garbage such as ransomware, crypo-miners, internet tracking, malware and the like.

Finally, let's also move to a secure DNS service whilst we are at it. Besides, [DNS over HTTPS is coming whether ISPs and governments like it or not]( https://nakedsecurity.sophos.com/2019/04/24/dns-over-https-is-coming-whether-isps-and-governments-like-it-or-not/). Firefox is slowly [enabling DNS over HTTPS](https://www.zdnet.com/article/mozilla-to-gradually-enable-dns-over-https-for-firefox-us-users-later-this-month/) so we can to!

## Pi-hole
What is a (the) Pi-hole? From the [documentation](https://docs.pi-hole.net/): 
> The Pi-hole® is a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.  
> - Easy-to-install: our versatile installer walks you through the process, and takes less than ten minutes 
> - Resolute: content is blocked in non-browser locations, such as ad-laden mobile apps and smart TVs 
> - Responsive: seamlessly speeds up the feel of everyday browsing by caching DNS queries 
> - Lightweight: runs smoothly with minimal hardware and software requirements 
> - Robust: a command line interface that is quality assured for interoperability 
> - Insightful: a beautiful responsive Web Interface dashboard to view and control your Pi-hole 
> - Versatile: can optionally function as a DHCP server, ensuring all your devices are protected automatically 
> - Scalable: capable of handling hundreds of millions of queries when installed on server-grade hardware 
> - Modern: blocks ads over both IPv4 and IPv6 
> - Free: open source software which helps ensure you are the sole person in control of your privacy

## Raspberry Pi Setup
Most home users run Pi-hole on Raspberry Pi hardware on top of a standard Raspian operating system install.  For details on setting up Raspian see [Installing operating system images](https://www.raspberrypi.org/documentation/installation/installing-images/) documentation.

## Non-Raspberry Pi Setup
As you can gather from the Pi-hole section above, Pi-hole does not *have* to be run on Raspberry Pi hardware. I have mine running in a VMware virtual machine running [Debian Server](https://www.debian.org/) (non-GUI) as it's base O/S. You could, for example install Debian and Pi-hole on any old x86 hardware you have laying about. To set Debian up for yourself, follow this excellent [howtoforge guide](https://www.howtoforge.com/tutorial/debian-minimal-server/).

For the purposes of this post, I'm going to setup Pi-Hole under Debian Buster in a VMware VM.  Regular readers will know that I use a [SpongeBob SquarePants](https://www.imdb.com/title/tt0206512/) naming scheme in my lab. This VM shall follow that convention and will have the hostname [mable](https://spongebob.fandom.com/wiki/Mable). :smile:

## DNS Over HTTPS - Cloudflared Argo Tunnel
Let's get DNS over HTTPS running.  To do this we are going to use Cloudflare's [Argo Tunnel](https://developers.cloudflare.com/argo-tunnel/reference/how-it-works/). We will use port 54 as our local endpoint of the Argo tunnel.
To install then:
```bash
wget https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-amd64.deb
dpkg -i cloudflared-stable-linux-amd64.deb
cloudflared --version
```
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Install Cloudflared" src="/images/pi-hole-https-dns/pi-hole-dns-https-03.png">

Next, let's test.  As the cloudflared daemon doesn't return to the command prompt once running, we need to test using screen:
```bash
sudo apt-get install screen
screen
sudo cloudflared proxy-dns --port 54 --upstream https://1.1.1.1/.well-known/dns-query --upstream https://1.0.0.1/.well-known/dns-query
```

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Tunnel Up" src="/images/pi-hole-https-dns/pi-hole-dns-https-04.png">

This will start a cloudflared daemon listening at port 54 for DNS requests which will then be forwarded over HTTPS to Cloudflare's 1.1.1.1 DNS service.

Hit Ctrl+A+D to leave the daemon and session running. Next step is to install Pi-hole and test.

## Installing Pi-hole
Installing Pi-hole involves just the one command. ([Alternative installation methods](https://github.com/pi-hole/pi-hole/#one-step-automated-install) are available):
```bash
curl -sSL https://install.pi-hole.net | sudo bash
```

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Install Pi-hole" src="/images/pi-hole-https-dns/pi-hole-dns-https-05.png">

Follow the setup process and complete any prompts along the way. It doesn't matter what default DNS we use at this point as we will be changing it to use the Argo Tunnel anyway. Make a note of the web interface URL and password. Once installed, confirm you can browse to the Pi-hole and login OK:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Pi-hole Admin" src="/images/pi-hole-https-dns/pi-hole-dns-https-06.png">

To change Pi-hole's web admin password:
```bash
pihole -a -p
```

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Pi-hole password change" src="/images/pi-hole-https-dns/pi-hole-dns-https-07.png">

## Configuring Pi-hole to use Cloudflared Argo Tunnel
Back at the command line, let's setup Pi-hole to use the Argo Tunnel via the cloudflared daemon started earlier. Enter the following:  
```bash
echo 'server=127.0.0.1#54' | sudo tee /etc/dnsmasq.d/02-pihole.conf
```
Final step is to comment out (place a '#' in front of) the two entries `PIHOLE_DNS_1=` and `PIHOLE_DNS_2=` in /etc/pihole/setupVars.conf:
```bash
sudo nano /etc/pihole/setupVars.conf
```
The above will configure pi-hole to use the cloudflared daemon listening at port 54 as its upstream DNS server. Save and restart pi hole with the command `pihole restartdns`

## Quick Test
Configure a machine to use your newly configured Pi-hole machine as it's DNS server.  Then browse to [dnsleaktest.com](https://www.dnsleaktest.com) and run the extended test. 
If all working correctly you should see Cloudflare listed as your ISP in the results:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="dnsleaktest results" src="/images/pi-hole-https-dns/pi-hole-dns-https-08.png">

You may have more than one DNS server listed, that's perfectly OK as long as they all list Cloudflare as your ISP :smile:

To confirm DNS over HTTPS functionality, browse to [https://1.1.1.1/help](https://1.1.1.1/help). The return should match below. Check that Using DNS over HTTPS (DoH) returns **Yes**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="cloudflare help results" src="/images/pi-hole-https-dns/pi-hole-dns-https-11.png">

## Run Cloudflared Argo Tunnel as a Service
Final piece is to configure Cloudflare to always run on startup. Edit the service unit file:
```bash
sudo nano /etc/systemd/system/dnsproxy.service
```
Paste in the following:
```bash
[Unit]
Description=CloudFlare DNS over HTTPS Proxy
Wants=network-online.target
After=network.target network-online.target

[Service]
ExecStart=/usr/local/bin/cloudflared proxy-dns --port 54 --upstream https://1.1.1.1/.well-known/dns-query --upstream https://1.0.0.1/.well-known/dns-query
Restart=always

[Install]
WantedBy=multi-user.target
```
Configure the service to auto start:
```bash
sudo systemctl enable dnsproxy.service
```
Reboot your pi-hole machine and confirm the newly created dnsproxy service starts and runs OK. Pi-hole too:
```bash
sudo service dnsproxy status
pihole status
```
<img style="display: block; margin-left: auto; margin-right: auto;" alt="dnsproxy status" src="/images/pi-hole-https-dns/pi-hole-dns-https-09.png">

## Blocklists
Now we cone to the fun part; blocking unwanted adverts and other unwanted stuff! :sunglasses:

There are plenty of sources of blocklists that can be used with Pi-hole. Here are some of my favourite sources:

- [firebog.net](https://firebog.net/) - Excellent source of blocklists curated by Wally3k
- [Block List Project](https://blocklist.site/) - Good source of blocklists free for non-commercial use
- [StevenBlack hosts](https://github.com/StevenBlack/hosts#unified-hosts-file-with-base-extensions) - Consolidated blocklists 
- [Pi-hole discourse thread](https://discourse.pi-hole.net/t/update-the-best-blocking-lists-for-the-pi-hole-alternative-dns-servers-2019/13620/57) - The best blocking lists for the Pi-Hole + Alternative DNS servers 2019

## Whitelists
The more you block, the more whitelists come into play.  Whitelists allow connectivity to domains even when they are listed in the blocklists.  Again, sources of whitelists are available on the internet:

- [firebog.net](https://firebog.net/) - Whitelists are towards the bottom of Wally3k's lists page
- [anudeepND whitelist](https://github.com/anudeepND/whitelist#commonly-white-listed-domains-for-pi-hole-compatible-with-pi-hole-docker-image) - Source of commonly whitelisted domains plus automated whitelist update tool
- [Pi-hole discourse thread](https://discourse.pi-hole.net/t/commonly-whitelisted-domains/212) - Whitelist discussion with plenty of suggestions

## Configuring Other Devices to use Pi-hole
This will vary from network to network depending on how devices attached to those networks obtain their network settings. Typically this will require a change on your internet router. Simply edit the local network DHCP settings that are pushed to network clients and enter the IP address of your pi-hole machine so that all clients use the pi-hole as their DNS server instead.

Pfsense has this setting in **Services - DHCP Server - LAN**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="pfsense DNS" src="/images/pi-hole-https-dns/pi-hole-dns-https-10.png">

## Conclusion
In this post we looked at setting up domain based blocking with the objective of keeping most types of internet bourne garbage (adverts, ransomware, crypo-miners, internet tracking, malware, etc) off of all devices connected to our network.  We achieved this by implementing Pi-hole and leveraging freely available blocklists.

As a bonus, we also enabled DNS over HTTPS to enhance privacy too.

Until next time, happy and safe web-surfing! :surfer:

-Chris


