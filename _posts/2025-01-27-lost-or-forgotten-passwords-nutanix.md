---
layout: post
title: "Resetting Lost or Forgotten Nutanix Passwords"
excerpt: "Yes, that is the correct password... Oh no it isn't!"
tags: 
- Nutanix
- Security
image:
  thumb: lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-01.png
comments: true
date: 2025-01-27T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Nutanix Security" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-01.png">
Passwords, passwords, passwords. We all have them, we all loose or forget them. Fact of life unfortunately. A quick post to detail methods to access your Nutanix environment and reset the missing password should you find yourself bereft of access for the given local account.

One post for them all if you like.

First off, a quick primer on creating an SSH public and private key pair. In some situations that I'll cover below, having an SSH key will allow us access to reset passwords. However the [standard disclaimer](/pages/disclaimer/){:target="_blank"} applies: With great power comes great responsibility: Handling and storage of SSH keys should be carefully considered too!
{% include _toc.html %}
## Creating an SSH Key Pair
To create an SSH private public key pair, the following command can be used. This command works in Linux, Windows, MacOS:
```
ssh-keygen -a 100 -t ed25519 -C "Chris @ PolarClouds"
```
Switches used:<br>
`-a 100` - 100 key derivation function rounds<br>
`-t ed25519` - Use Ed25519 cryptography, creates secure and compact keys<br>
`-C "Chris @ PolarClouds"` - Comment, allows tracking of the keys<br>

For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Create SSH Key pair - Windows" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-02.png">

Use of a passphrase is optional. Think of it as a password for your SSH key. 

As returned by the command, the keys are stored in `C:\Users\Administrator\.ssh`. 
The private key has no file extension, where as the public key has the .pub extension. Lets take a look and open the public key:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="SSH Key pair - Windows" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-03.png">
*(Don't worry, these keys have already been deleted!)*

## Importing SSH Keys in to a Nutanix Environment
Next, lets import our new public key into our Nutanix environment:<br>
1. Log onto your cluster Prism Element interface
2. Select **Settings (Cog in top right hand corner) > Cluster Lockdown**
3. (Optional) Repeat for Prism Central. After logging into Prism Central, select **Settings (Cog in top right hand corner) > More Settings > Cluster Lockdown**
4. Name the key and paste the contents of the .pub key in to the key space:<br><br>
  <img style="display: block; margin-left: auto; margin-right: auto;" alt="Import SSH key into Prism" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-04.png"><br>
5. Save when done. Lets test from the machine I used to create the key pair:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="SSH key test" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-05.png">

Working nicely. As a bonus, as I did not use a passphrase when creating my keys, I don't get prompted for it, I'm able to access with zero prompting.

As long as the public key remains imported into Prism Element and/or Prism Central and both exist on whatever machine I use to access the required environment, the keys will be used for access.

There is no need to create multiple keys for multiple password resets; the same key can be used any number of times. 

If I want to log in from other machines, I'll need to copy the key pair to those machines too. Alternatively, I may want to store the key pair in a password safe for emergency "break glass" Prism Element and Prism Central access.

### Lights Out Card Admin Password - AHV Host (IPMI / ILO / IDRAC / CIMC)
Use the ipmitool utility built into AHV to reset the lights out password.
1. Open ssh session to AHV using root account:<br>
  ```
  ssh root@<AHV IP ADDRESS>
  ```
2. To find the user ID of the administrator for which you want to change the password:
  ```
  ipmitool user list 1
  ```
3. To reset the password using the found user ID:
  ```
  ipmitool user set password <USER ID> <NEW PASSWORD>
  ```

For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Lights Out Example" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-06.png">

### root Password - AHV Host
Using a SSH key allows us to access an AHV host as root without knowing the password. We can then reset the password.
1. Create an SSH key pair - follow [Creating an SSH Key pair](/lost-or-forgotten-passwords-nutanix/#creating-an-ssh-key-pair) above
2. Import the public SSH key into Prism Element - follow [Importing SSH Keys in to Nutanix Environment](/lost-or-forgotten-passwords-nutanix/#importing-ssh-keys-in-to-nutanix-environment) above
3. From the machine used to create the key pair, SSH as the nutanix user to the CVM running **on the affected AHV host**:
  ```
  ssh nutanix@<CVM IP ADDRESS>
  ```
4. From inside the CVM SSH session established in step 3, SSH to the AHV host using the CVM to Hypervisor network: 
  ```
  ssh root@192.168.5.1
  ```
5. Reset the root password using the command:
  ```
  passwd root
  ```

For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Change AHV root Password" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-07.png">

If you need to change the root password on all the AHV hosts belonging to a cluster, the following script can be run on any CVM to save the time it takes to log on to all the AHV hosts individually: 
  ```
echo -e "CHANGING ALL AHV HOST ROOT PASSWORDS.\nPlease input new password: "; read -rs password1; echo "Confirm new password: "; read -rs password2; if [ "$password1" == "$password2" ]; then for host in $(hostips); do echo Host $host; echo $password1 | ssh root@$host "passwd --stdin root"; done; else echo "The passwords do not match"; fi
  ```
See the Solution section of [KB-6153](https://portal.nutanix.com/kb/6153){:target="_blank"} for details.

### nutanix Password - Cluster / Prism Element
Using a SSH key allows us to access any CVM in a cluster as nutanix without knowing the password. We can then reset the password.
1. Create an SSH key pair - follow [Creating an SSH Key pair](/lost-or-forgotten-passwords-nutanix/#creating-an-ssh-key-pair) above
2. Import the public SSH key into Prism Element - follow [Importing SSH Keys in to Nutanix Environment](/lost-or-forgotten-passwords-nutanix/#importing-ssh-keys-in-to-nutanix-environment) above
3. From the machine used to create the key pair, SSH as the nutanix user to any CVM of the affected cluster:
  ```
  ssh nutanix@<ANY CVM IP ADDRESS>
  ```
4. Reset the nutanix password using the command:
  ```
  sudo passwd nutanix
  ```

For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Change cluster nutanix Password" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-08.png">

### admin Password - Cluster / Prism Element
We can simply use the cluster's nutanix user to change the cluster's admin password (no need for SSH keys):
1. SSH as the nutanix user to any CVM of the affected cluster:
  ```
  ssh nutanix@<ANY CVM IP ADDRESS>
  ```
4. Reset the admin password using the command:
  ```
  sudo passwd admin
  ```

For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Change cluster admin Password" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-09.png">

If you also need to unlock the account:
  ```
  sudo faillock --user admin --reset
  ```
 
### admin Password - Prism Central
We can use the Prism Central VM console and nutanix user to change the Prism Central admin password:

1. Log into Prism Element of a cluster running Prism Central (PC) VM(s) 
2. Open a PC VM console (Right click VM name and select **Launch Console**)
3. Log in as the Prism Central nutanix user
4. Reset the admin password using the command: 
  ```
  sudo passwd admin
  ```

For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Change Prism Central admin Password" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-10.png">

If you also need to unlock the account:
  ```
  sudo faillock --user admin --reset
  ```

### nutanix Password - Prism Central
We can use the Prism Central VM console and admin user to change the Prism Central nutanix password:

1. Log into Prism Element of a cluster running Prism Central VM(s) 
2. Open a PC VM console (Right click VM name and select **Launch Console**)
3. Log in as the Prism Central admin user
4. Reset the nutanix password using the command:
  ```
  sudo passwd nutanix
  ```
5. Re-enter the admin password for verification

For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Change Prism Central nutanix Password" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-11.png">

## Conclusion and Wrap Up
There we have it; methods to reset the six most important local passwords in any Nutanix deployment.

As external identity providers are supported, these should ideally be used for day to day environment access, with local accounts only used in emergency situations.

Take a look at the [Identity and Access Management (IAM) section of the Nutanix Security Guide](https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Security-Guide-v7_0:mul-iam-introduction-c.html){:target="_blank"} for details of supported external identity providers and provider configuration.

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Oprah Password" src="/images/lost-or-forgotten-passwords-nutanix/lost-or-forgotten-passwords-nutanix-12.jpg">

Thanks Oprah! 

-Chris