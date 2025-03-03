---
layout: post
title: "Migrating Azure VMs to Nutanix"
excerpt: "Elevate your cloud game: Discover the simple and efficient way to migrate Azure VMs to Nutanix"
tags: 
- Nutanix
- Azure
image:
  thumb: migrate-azure-nutanix/migrate-azure-nutanix-00.png
comments: true
date: 2025-03-03T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Migration" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-00.png">
As organisations continue to evolve their IT infrastructure, the need for seamless migration and management of virtual machines (VMs) across different cloud environments has become increasingly important. In this blog post, we'll be exploring the process of migrating Azure VMs to Nutanix, a leading hyperconverged infrastructure (HCI) solution that enables you to run any workload on any public cloud or on-premises environment with ease.

Migrating Azure VMs to Nutanix can seem daunting at first, but with the right tools and knowledge, it's a straightforward process. In this post, we'll walk through the steps involved in migrating Azure VMs to Nutanix, including choosing the VMs you want to migrate, selecting the target container, and understanding the migration considerations that come into play.

Whether you're looking to reduce costs by moving workloads from public cloud providers like Microsoft Azure to on-premises infrastructure or wanting to take advantage of the scalability and flexibility offered by Nutanix's Cloud-First Disaster Recovery solution, this post will provide a comprehensive overview of the process involved in migrating Azure VMs to Nutanix. So let's dive in!

{% include _toc.html %}
## Introducing Nutanix Move
Nutanix Move is a powerful migration tool that enables you to seamlessly move VMs between different environments, including on-premises infrastructure and public cloud providers like Microsoft Azure:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Nutanix Move Capability" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-00a.png">

As discussed, in this post we will migrate an Azure-native VM to a Nutanix private cloud. 

## Move Deployment
For the purpose of this post, we will be focusing on the migration process itself. Information regarding the deployment of Nutanix Move can be found at:

- My PolarClouds post [Quickly Migrate from Free ESXi to Nutanix Community Edition](/migrate-from-free-esxi-to-nutanix-community-edition/){:target="_blank"}, specifically the section [Installing and Configuring Move](/migrate-from-free-esxi-to-nutanix-community-edition/#installing-and-configuring-move){:target="_blank"}

- Nutanix [Move User Guide](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Move-v5_5:Nutanix-Move-v5_5){:target="_blank"}, specifically the section [Deploying Move](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Move-v5_5:top-deploy-alt-t.html){:target="_blank"}

Following best practice, we will deploy the Move appliance onto our target environment, in this case our Nutanix infrastructure.

## Registering Move App in Azure and Applying Privileges
To be able to export VMs from Azure, we need to grant Move access to our source Azure environment. This can be completed following the steps outlined in the documentation, specifically the section [Azure to AHV](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Move-v5_5:top-azure-ahv-migration-c.html){:target="_blank"}. Also documented there are migration considerations, requirements and limitations. 

For the purposes of this post, we will use the Move appliance command line (CLI) method to register the Move application in Azure and apply the required privileges.

From an SSH session to our newly deployed Move appliance, Switch to the root user and launch `create-azure-app`:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="SSH to Move" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-00b.png">

After logging in and selecting my Azure subscription, the CLI prompts and my responses are as follows:
{% highlight shell %}
Please enter a name to be used with the app. Press 'Enter' for default (Default: NutanixMoveApp): NutanixMoveApp
Please enter a name for the custom role. Press 'Enter' for default (Default: Nutanix Move Operator): Nutanix Move Operator
Please enter the type of role to be assigned to the app. Options are (Source,Target,Both). Press 'Enter' for default (Default: Both): Both
Do you want to add public IP permissions to create and assign Public IP address to VMs on Azure target? (y/n). Press 'Enter' for default (Default: n): y
{% endhighlight %}

After which the Nutanix Move Azure app is created and assigned with the custom role. 

Finally, the CLI provides the following information which can be used in the Move GUI to register the Azure source environment: 

- Subscription ID
- Tenant ID
- Client ID
- Client Secret

Unfortunately, using Move 5.5.2 and Azure in mid February 2025, the following error can be observed when combing through the create-azure-app console output:
{% highlight shell %}
Adding the custom role 'Nutanix Move Operator' to the app...
ERROR: the following arguments are required: --scope
Examples from AI knowledge base:
az role assignment create --assignee sp_name --role a_role
Create role assignment for an assignee.  
{% endhighlight %}

Subsequently trying to add the Azure Environment to Move

*Azure operation 'Authorization' failed. User does not have the required permissions as per the permission policy specified*:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Error" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-01.png">

Yep, Microsoft have changed the requirements of the `az role assignment` command... :unamused:

Nutanix Engineering are aware of the issue and this will be fixed in upcoming versions (v5.5.3+).

### Add Role Assignment 
Luckily resolving this is simple enough. 

Log into the Azure Portal, select **Subscriptions > Your Subscription > Access control (IAM) > Add role assignment**.

Select `Nutanix Move Operator` role, click **Members > +Select Members**, search for and select `NutanixMoveApp`

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Fix" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-02.png">

Finally, click **Review + assign** at the bottom of the wizard.

After retrying the registration in the Move GUI, Azure registration completed without issue:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Move Environments" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-03.png">

## Azure to Nutanix: Migration Configuration
So let's do this. I'll create a new Migration Plan:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create Plan" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-04.png">

I'll select my source and target environments:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Move Source and Target" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-05.png">

I'll select my source VM:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Move Source VM" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-06.png">

Quick look at the warning (!), yep that's no problem, I'm running AOS v7.0 on my target cluster:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Move Source VM Warning" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-07.png">

I'll select my target and optional test networks:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Target Networks" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-08.png">

I'll tick **Allow Nutanix Move to add CD-ROM on the target VMs** and accept all other defaults:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Preparation" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-09.png">

I'll accept the default VM Settings:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="VM Settings" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-10.png">

The summary of my Azure to Nutanix VM migration looks like this:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Move Migration Summary" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-11.png">

## Azure to Nutanix: The Migration

After clicking **Save and Start**, Move will begin the migration. During this time my Azure VM remains online and available to production workloads:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Move Migration Preparation" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-12.png">

Next, Move will seed my Azure VM data into my Nutanix environment. Again, my Azure VM is online throughout:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Move Data Seeding" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-13.png">

With seeding complete, I'm ready to cutover my Azure VM to Nutanix. Let's click the Cutover button:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Ready to Cutover" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-14.png">

As per the warning, after proceeding with the cutover, my Azure VM will be shutdown and final changes will be replicated to my Nutanix environment: 

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Begin Cutover" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-15.png">

Final data synchronisation in progress:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Final Data Sync" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-16.png">

With the synchronisation complete, a Nutanix VM is created, assigned the replicated data and powered on:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Power VM On" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-17.png">

Success! My Azure VM is now running on my Nutanix environment:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Success" src="/images/migrate-azure-nutanix/migrate-azure-nutanix-18.png">

After verifying that my applications are functioning as expected and services have been resumed, I can safely delete the shutdown Azure VM. Note that Move leaves the source Azure VM in a shut down state after migration.

Alternatively, if I discover issues related to the migration or I simply want to back out the migration for whatever reason, I can shutdown my Nutanix VM and power my source Azure VM back on. 

## Conclusion and Wrap Up
In this post, I walked you through the process of migrating Azure VMs to Nutanix using Nutanix Move. From preparing your source and target environments to verifying application functionality after migration, we can ensure a smooth transition each and every time. By following these instructions, you'll be able to elevate your cloud game by leveraging the benefits of hybrid cloud computing with Nutanix.

As seen in the graphic above in the section [Introducing Nutanix Move](/migrate-azure-nutanix/#introducing-nutanix-move), should you not want to run Nutanix on-prem, the same process can be used to migrate your Azure VMs to Nutanix Cloud Clusters (NC2) running on public cloud. Find out more about NC2 Azure at [Microsoft Learn](https://learn.microsoft.com/en-us/azure/baremetal-infrastructure/workloads/nc2-on-azure/about-nc2-on-azure){:target="_blank"} or more generally at [Nutanix UK](https://www.nutanix.com/uk/products/nutanix-cloud-clusters){:target="_blank"}.

As organisations continue to evolve their IT infrastructure and adopt hybrid cloud strategies along with cost savings, migrations like this are becoming increasingly common. By leveraging tools like Move and understanding the technical nuances involved in migration, we can ensure successful transitions that minimise costs and downtime and maximise application availability.

-Chris