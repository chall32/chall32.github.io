---
layout: post
title: "Nutanix Prism Central Deployment" 
excerpt: "Multi Cluster Management and More"
tags: 
- Hyper-V
- Deployment
- ESXi
- Nutanix
image:
  thumb: nutanix-prism-central-deployment/nutanix-prism-central-deployment-00.png
comments: true
date: 2023-06-21T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Nutanix and ESXi" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-00.png">
Continuing with our series looking at Nutanix, today we will take a look at Prism Central and its deployment.

First off, for my non-Nutanix familiar readers, what is Prism Central?

In much the same way as VMware vCenter is a separate application that can be deployed to manage multiple VMware vSphere clusters, Prism Central (PC) is a separate application that can be deployed to manage multiple Nutanix clusters. 

Much like vCenter and vCenter High Availability, Prism Central can be deployed as a single virtual machine or in a three node "scale out" cluster. The scale out PC deployment option can provide additional resiliency in environments that require it.

Unlike vCenter, Prism Element - the default host management plane deployed when you deploy a Nutanix host and it's control VM - is still required to manage some lower level configuration of hosts and VMs.

Having said that, also unlike vCenter, Prism Central is also used to enable and manage the following products:

- Flow (Virtual Networking and Micro-segmentation)
- Calm (Automation)
- Files (SMB and NFS File Services)
- Foundation Central (Cluster Deployment)
- Karbon (Kubernetes) 
- Objects (S3 Compatible Storage)


Finally, both VM protection and VM efficiency analysis are also built into Prism Central, rather than requiring a separate products and licences such as VMware Site Recovery Manager for VM protection and VMware vRealise (or Aria as it is now called) for VM efficiency monitoring. 

Looking VM efficiency in PC for example, we can see:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM efficiency" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-23.png">

Nice! 

Not only that, Prism Central can also perform [full capacity planning and reporting](https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2023_1_0_1:mul-resource-planning-pc-c.html){:target="_blank"} out of the box.

Although we are getting ahead of ourselves. Let's deploy Prism Central already!
{% include _toc.html %}
## Prism Central Deployment
From our community edition deployment, select the **Register or create new** option:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 1" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-01.png">
New please:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 2" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-02.png">

Let's select the latest compatible version:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 3" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-03.png">

As this is a test lab, let's go with a single VM PC deployment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 4" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-04.png">

Again, as this is a lab, let's go with the small option. We will also configure networking and storage:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 5" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-05.png">

As well as the hostname and IP address of our PC VM:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 6" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-06.png">

Confirm configuration and click **Deploy**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 7" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-07.png">

Monitoring the deployment via the Prism Element tasks view: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 8" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-08.png">
A quick look at the newly deployed PC VM console: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 9" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-09.png">
Finally, after deployment, let's browse to our Prism Central VM:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 10" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-10.png">
As was seen when deploying our host, Chrome does not like the out of the box SSL certificate. Easily fixed by clicking on the browser window and typing:
{% highlight shell %}
thisisunsafe
{% endhighlight %}

The default credentials are:
- Username: **admin**
- Password: **Nutanix/4u**

Let's create a new password:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 11" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-11.png">
Continue with Pulse telemetry enabled:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 12" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-12.png">
Deployment complete!

What's more is that we have 90 days of Prism Central Ultimate out of the box too:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 13" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-13.png">
Back in Prism Element, let's connect our cluster to our newly deployed Prism Central. Select the **Register or create new** option again and select **Connect**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 14" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-14.png">

Click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 15" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-15.png">

Supply our newly deployed Prism Central details and **Connect**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 16" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-16.png">

Again we can monitor the connection process via the Prism Element tasks view: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 17" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-17.png">
Back in Prism Central we can see that our cluster connection is complete. Notice that our cluster SITE-A-CLUSTER listed: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 18" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-18.png">
## Web SSL Certificate Replacement
Finally, let's fix our web SSL certificate to keep Chrome happy.

Click the hamburger menu (top left hand corner), scroll down to and select **Prism Central Settings**, select **SSL Certificate** and **Replace Certificate**:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 19" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-19.png">
As this is a lab, I'll go with regenerating another self signed certificate:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 20" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-20.png">
Allow time for the certificate to be generated imported into Prism Central. A refresh and Chrome now gives us the option to proceed: 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 21" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-21.png">
Job done! Prism Central deployed and managing our cluster:
<img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Deploy 22" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-22.png">

## Find Out for Yourself
Don't just take my word for it, have a look and play with a Prism Central instance for yourself at [demo.nutanix.com](https://demo.nutanix.com){:target="_blank"}:

<a target="_blank" href="https://demo.nutanix.com"><img style="display: block; margin-left: auto; margin-right: auto;" alt="PC Demo" src="/images/nutanix-prism-central-deployment/nutanix-prism-central-deployment-24.png"></a>
<p style="text-align: center;">(Click image above to go to demo.nutanix.com)</p>

## Conclusion and Wrap Up
A high level look at Nutanix Prism Central, it's capabilities and it's deployment.

As opposed to the VMware model where each service requires it's own management console - for example VMware NSX manager for virtual networking and micro-segmentation, VMware SRM manager for VM protection; Nutanix Prism Central provides a "one stop shop" for these services and more - built in. 

For those that need it the [Prism Central Documentation 'stack'](https://portal.nutanix.com/page/documents/list?type=software&filterKey=software&filterVal=Prism){:target="_blank"} is available publicly without the need for a logon.


Finally, finally, given [recent news](https://www.reuters.com/markets/deals/eu-antitrust-regulators-okay-broadcom-vmware-deal-sources-says-2023-06-12/){:target="_blank"} would not now be a great time to start looking at alternatives?

-Chris