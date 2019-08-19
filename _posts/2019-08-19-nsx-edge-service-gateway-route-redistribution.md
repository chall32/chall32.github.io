---
layout: post
title: NSX Edge Service Gateway Route Redistribution
excerpt: Option to push BGP/OSPF routes northbound greyed out? No problem!
tags:
- VMware
- Pro-Tip
image:
  thumb: nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-00.png
comments: true
date: 2019-08-19T13:00:00+00:00
---
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Which Route?" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-01.png">
<span class="image-credit" style="float: right; margin: 0px 0px 10px 10px;">Photo: <a href="https://unsplash.com/@soymeraki?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Javier Allegue Barros</a></span>

Deploying VMware NSX-V 6.4.5 from scratch into production in an active/active/active mode, (yep three sites!) we ran into an interesting problem when looking at the configuration of the Edge Service Gateway (ESG) on the secondary sites.

Can you spot it in the screenshot from my test lab below?
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Route Redistribution greyed out" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-02.png">

Let me give you a clue:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Zoom Route Redistribution greyed out" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-03.png">

Yes thats correct, the option to enable OSPF / BGP route redistribution from the ESG is greyed out!

Let's check the flash client:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Route Redistribution greyed out Flash too" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-04.png">

Yep same :(

Okay, so that might not be a problem when running NSX-V in active/passive mode, but we are trying to run active/active/active here - I.E. run active services from all three datacentres.

Here is how to fix. It involves talking to NSX at an API level, but stick with me, its an easy fix.

First off, download a copy of Postman from [getpostman.com](https://www.getpostman.com/downloads/) and install.

Once installed, we need to configure Postman to work with NSX, so close Postman's getting started screen and select **File - Settings - General - SSL Certificate Verification** is set to off:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Postman SSL verification off" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-05.png">

Next, select Proxy and ensure **Global Proxy Configuration** and **Use System Proxy** are both set to off:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Postman System and Global Proxy off" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-06.png">

Close Postman configuration.  

Next select **Get** from the drop down, **Basic Auth** from the **Authorisation** drop down and enter credentials to your secondary NSX Manager as shown below:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Postman Get Config 1" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-07.png">

Select **Headers**, set Key to **Content-Type**, Value to **application/xml** and enter the following URL (modify to match your environment): **https://FQDN_of_Secondary_NSX_Manager/api/4.0/edges/edge-ID/routing/config**

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Postman Get Config 2" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-08.png">

Click **Send**

Your results should fill with xml similar to the below.  If not, check your NSX Manager FQDN, NSX credentials and Edge ID.

Click to copy results to clipboard:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX API results" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-09.png">

Paste results into a text editor such as [Notepad++](https://notepad-plus-plus.org/)

Find the XML section between `<redistribution>` and `</redistribution>` headings. 

Replace the whole `<redistribution>` section with the following:
```
        <redistribution>
            <enabled>true</enabled>
            <rules>
                <rule>
                    <id>0</id>
                    <from>
                        <ospf>false</ospf>
                        <bgp>true</bgp>
                        <static>true</static>
                        <connected>true</connected>
                    </from>
                    <action>permit</action>
                </rule>
           </rules>
        </redistribution>
```
Modify ospf, bgp, static and connected sections to match your requirements, such as those set on your ESGs at your primary site.

Once complete, open a new tab in Postman, set type to **Put**, select **Headers**, set Params Key to **Content-Type**, Value to **application/xml** and enter URL:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX API Put 1" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-10.png">


Set authorisation to **Basic Auth**:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX API Put 2" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-11.png">

Select **Body** and **Raw**. Paste modified xml into window
<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX API Put 3" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-12.png">
Finally, click **Send**.

Confirm NSX returns a **204 No Content** return to Postman:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX API Put return" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-13.png">

If not, retry GET, xml modification and PUT again.  Pay close attention to `<version>` tags in the received and sent xml; they must match.

Refresh NSX to confirm modification has applied.  Sure the configuration is still greyed out, but it's enabled now:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="NSX Secondary ESG Route Redistribution" src="/images/nsx-edge-service-gateway-route-redistribution/esg-route-redist-fix-14.png">

Done!  Repeat for any other ESG's at any other secondary sites :)

VMware engineering have confirmed this is an issue with NSX-V 6.4.5. *Should* be fixed in NSX-V 6.4.6.

-Chris





