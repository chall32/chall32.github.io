---
layout: post
title: "Nutanix Cluster Profiles"
excerpt: "Consistent Cross Cluster Configuration Compliance"
tags:  
- Nutanix
image:
  thumb: nutanix-cluster-profiles/nutanix-cluster-profiles-00.png
comments: true
date: 2025-09-09T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Nutanix Cluster Profiles" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-00.png">
One of the new Prism Central 2024.3.x features that may have escaped your notice (it did mine), is the addition of Cluster profiles.

Cluster profiles allow you to deploy and manage unified configurations across multiple clusters from Prism Central, ensuring consistency whilst tracking and allowing for correction of any configuration drift. Sure other virtualisation products have had host profiles available for a while now... :wink:

Let's look at creating a cluster profile using Prism Central 7.3 (yes, [new product versioning scheme](https://portal.nutanix.com/page/documents/details?targetId=Release-Notes-Prism-Central-vpc_7_3:Release-Notes-Prism-Central-vpc_7_3){:target="_blank"}) and applying it to a cluster. 

In Prism Central, navigate to **Infrastructure - Hardware - Clusters - Profiles**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Welcome" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-01.png">

Click **Create Profile** and name the profile:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Creation 1" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-02.png">

I'll **Configure new settings values** and click **Next**:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Creation 2" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-03.png">

Next, configure all available settings. As per the pop-up, more settings will be available soon.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Creation 3" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-04.png">

For example, adding DNS servers is simple enough:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Creation 4" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-05.png">

I won't allow override as I want to track out of compliance issues:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Creation 5" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-06.png">

Next, I'll select my profile and assign it to a cluster:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Creation 6" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-07.png">

Checking Tasks, profile assigned successfully: 

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Creation 7" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-08.png">

Yep, looking good:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cluster Profiles Creation 8" src="/images/nutanix-cluster-profiles/nutanix-cluster-profiles-09.png">

So there we have it, Cluster profiles. Configure once and apply to multiple clusters. Simple!

You can find the documentation on this feature here: [Prism Central Infrastructure Guide](https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_7_3:mul-settings-profiles-clusters-pc-c.html){:target="_blank"}.

-Chris