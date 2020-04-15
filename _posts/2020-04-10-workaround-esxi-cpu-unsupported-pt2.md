---
layout: post
title: "Workaround ESXi CPU Unsupported Error - Part 2" 
excerpt: "CPUID and EAX Between Friends"
tags: 
- Free
- Pro-Tip
- VMware
- ESXi
image:
  thumb: workaround-esxi-cpu-unsupported/workaround-esxi-cpu-unsupported-00.png
comments: true
date: 2020-04-10T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="vSphere Logo" src="/images/workaround-esxi-cpu-unsupported/workaround-esxi-cpu-unsupported-00.png">
Last time we looked at a workaround to install ESXi 7.0 into a VMware virtual machine hosted on physical hardware that contains an unsupported CPU. If you've not seen that post, [catch up now](https://polarclouds.co.uk/workaround-esxi-cpu-unsupported/). It's a great read. 

As mentioned, this post is part 2 of a multipart series.  Find the other parts here:

-  Part 1: [Be gone CPU_SUPPORT Error!](https://polarclouds.co.uk/workaround-esxi-cpu-unsupported/)
-  Part 2: This part (CPUID and EAX Between Friends)
-  Part 3: [Lets Get Virtual to Physical](https://polarclouds.co.uk/workaround-esxi-cpu-unsupported-pt3/)

To recap, the ESXi installer uses the CPUID instruction to identify the CPU(s) installed in the system. From the value obtained the user is told that their processor is either:
-  **Unsupported**. At which point the installer quits. No ESXi 7.0 for you!
-  **Will be unsupported in a later ESXi version**. The installer will allow install continuation.
-  ***Nothing***. The installer accepts that a valid CPU is present and silently continues the installation.

{% include _toc.html %}
## CPUID and the EAX Value
The CPUID instruction returns processor identification and feature information held in the EAX registers of the CPU in the system.

You can read more about the construction of the EAX value at [x86 Instruction Set Reference - CPUID](https://c9x.me/x86/html/file_module_x86_id_45.html) and [CPUID — CPU Identification](https://www.felixcloutier.com/x86/cpuid).

## Determining a CPUID 
As luck would have it, Intel publish CPUID's of their processors (or Processor Signatures as some of their literature calls it) in [Intel Architecture and Processor Identification With CPUID Model and Family Numbers](https://software.intel.com/en-us/articles/intel-architecture-and-processor-identification-with-cpuid-model-and-family-numbers) and [Intel Xeon Processor Scalable Family Update March 2020](
https://www.intel.com/content/dam/www/public/us/en/documents/specification-updates/xeon-scalable-spec-update.pdf).

Let's put that aside for the moment and single step through determining the CPUID of processor.

### 1. Find the Processor Model Number
Simple.  Check in **Configure - Hardware - System** of the host system that is to run the ESXi 7.0 VM. For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Processor Model Number" src="/images/workaround-esxi-cpu-unsupported-pt2/workaround-esxi-cpu-pt2-02.png">

In my case, my ESXi 6.7 host is running E5640 CPUs.

### 2. Use CPU-World
Next, use the search function of [CPU-World](http://www.cpu-world.com/) to obtain further information on the processor model found in step 1. For example:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="CPU_World CPUID" src="/images/workaround-esxi-cpu-unsupported-pt2/workaround-esxi-cpu-pt2-03.png">

Looks like the E5640 CPUs have a CPUID of 206C2h (h = hex notation), so 206C2.

### 3. Cross Check
Lets check the [Intel Architecture and Processor Identification With CPUID Model and Family Numbers](https://software.intel.com/en-us/articles/intel-architecture-and-processor-identification-with-cpuid-model-and-family-numbers), paper from above and sure enough, we can see that Gulftown and Westermere-EP CPU's have and CPUID of 0x206Cx (the "0x" prefix = hex notation, "x" suffix = variable):

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Westmere-EP CPUID" src="/images/workaround-esxi-cpu-unsupported-pt2/workaround-esxi-cpu-pt2-01.png">

So yes, 206C2 is a valid CPUID.

### 4. The Quick Way 
Connect via SSH to your ESXi host and issue the command
`esxcli hardware cpu cpuid raw list -c 0`

<img style="display: block; margin-left: auto; margin-right: auto;" alt="CPUID via SSH" src="/images/workaround-esxi-cpu-unsupported-pt2/workaround-esxi-cpu-pt2-04.png">

A confirmed CPUID of 206C2 padded with three zeros! Nice :smile:

## Converting a CPUID to an EAX Value
As we saw in [Workaround ESXi CPU Unsupported Error - Part 1](https://polarclouds.co.uk/workaround-esxi-cpu-unsupported/), an EAX number must be entered into the vSphere CPU Identification Mask setting in binary:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="CPU Advanced Settings" src="/images/workaround-esxi-cpu-unsupported/workaround-esxi-cpu-unsupported-04.png">

Conversion is easy.  Use an online [hexadecimal to binary converter](https://www.binaryhexconverter.com/hex-to-binary-converter). 

Again, using the three zero padded 000206C2 example from above:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="binary from hex" src="/images/workaround-esxi-cpu-unsupported-pt2/workaround-esxi-cpu-pt2-05.png">

The binary EAX value we need to enter to pass through to the VM a CPUID of a Westermere-EP CPU is `0000:0000:0000:0010:0000:0110:1100:0010` when using colons instead of spaces, as vSphere expects it.

## Pulling It All Together
So I present to you a list of processors, their CPUIDs, their binary EAX numbers and the result of installing ESXi 7.0 into a VM configured with the corresponding EAX number.:
<div>
<style scoped>
table{
    margin: 0 auto;
    width: 70%;
    border-collapse: collapse;
    border-spacing: 0;
    border:1px solid #000000; }
th{
    text-align: center;
    border:1px solid #000000; }
td{
    text-align: center;
    border:1px solid #000000;}
tr:nth-child(even) {
    background-color: #efefef;}
</style>
</div>
### Intel CPUIDs and Binary EAX Values

| Processor Generation | CPUID   | Binary EAX Value                | ESXi 7.0 Installer <br>Action  |
|:--------------------:|:-------:|:------------------------:|:--------------------------:|
| Westmere-EP          | 206C2 | `0000:0000:0000:0010:0000:0110:1100:0010` | <span style="color:red">Fail</span>    |
| SandyBridge          | 206A2 | `0000:0000:0000:0010:0000:0110:1010:0010` | <span style="color:blue">Warning</span> |
| IvyBridge 	       | 306A2 | `0000:0000:0000:0011:0000:0110:1010:0010` | <span style="color:blue">Warning</span> |
| Haswell              | 306C3 | `0000:0000:0000:0011:0000:0110:1100:0011` | <span style="color:green">**PASS**</span> |
| Broadwell            | 406F1 | `0000:0000:0000:0100:0000:0110:1111:0001` | <span style="color:green">**PASS**</span> |
| Skylake	           | 50654 | `0000:0000:0000:0101:0000:0110:0101:0100` | <span style="color:green">**PASS**</span> |
| Kabby Lake           | 806E9 | `0000:0000:0000:1000:0000:0110:1110:1001` | <span style="color:green">**PASS**</span> |

<br>
Eagle eyed readers will recognise that in [Part 1](https://polarclouds.co.uk/workaround-esxi-cpu-unsupported/), we used the Haswell processor EAX value to enable ESXi 7.0 installation. :stuck_out_tongue_winking_eye:

### AMD CPUIDs and Binary EAX Values

| Processor Generation | CPUID   | Binary EAX Value                | ESXi 7.0 Installer <br>Action  |
|:--------------------:|:------:|:------------------------:|:--------------------------:|
| Opteron 6124HE       | 100F91 | `0000:0000:0001:0000:0000:1111:1001:0001` | Untested*  |
| Opteron 6212	       | 600F12 | `0000:0000:0110:0000:0000:1111:0001:0010` | Untested*  |
| Opteron 6320	       | 600F20 | `0000:0000:0110:0000:0000:1111:0010:0000` | Untested*  |
| Epyc 7251	           | 800F12 | `0000:0000:1000:0000:0000:1111:0001:0010` | Untested*  |
| Epyc 7371		       | 800F12 | `0000:0000:1000:0000:0000:1111:0001:0010` | Untested*  |

<br>
*As server used for testing is Intel CPU based, I'm unable to test EAX values for AMD CPUs.

The environment used for the above testing:

-  Server: Dell R710, fitted with two Xeon Westmere-EP CPUs
-  Host ESXi: ESXi 6.7 build 15160138 installed
-  Managed by: vCenter 6.7 build 15129973
-  Test VM compatibility: ESXi 6.7 Update 2 and later (VM version 15)
-  Test VM guest OS: VMware ESXi 6.5 or later
-  ESXi 7.0 installer: VMware-VMvisor-Installer-7.0.0-15843807.x86_64.iso


## Conclusion and Wrap Up 
In this post we looked at CPUIDs, how to obtain them from online sources and how to find them by querying physical hardware. From there we looked at how to convert a CPUID into a binary EAX value.

Finally, we applied the generated EAX values to a VM and ran the ESXi 7.0 installer, with results as detailed above.

Next time, in part 3, we'll look at using what we've learnt in parts 1 and 2 in the physical world.  *Stay tuned..!*  :smiley: :computer:

-Chris