---
layout: post
title: "ESXi 7.0: The Unsupported" 
excerpt: "The Unconfirmed Missing"
tags: 
- Free
- Pro-Tip
- VMware
- ESXi
image:
  thumb: esxi7-missing-percs/esxi7-missing-percs-00.png
comments: true
date: 2020-04-29T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Controller" src="/images/esxi7-missing-percs/esxi7-missing-percs-00.png">
In researching for a solution to my recent troubles trying to get the storage controller in my home server seen when booted into ESXi 7.0 (see [The Missing PERC(s)](https://polarclouds.co.uk/esxi7-missing-percs/) for the full details), I happened to download a copy of the Dell customised ESXi 7.0 ISO file VMware-VMvisor-Installer-7.0.0-15843807.x86_64-DellEMC_Customized-A00.iso via [this guide](https://www.dell.com/support/article/en-us/sln288152/how-to-download-the-dell-customized-esxi-embedded-iso-image?lang=en).


Upon looking in \UPGRADE\PRECHECK.PY file, I tripped over a veritable "treasure trove" list of devices that are not supported due to the [VMKLinux Driver Stack Deprecation](https://blogs.vmware.com/vsphere/2019/04/what-is-the-impact-of-the-vmklinux-driver-stack-deprecation.html) in ESXi 7.0.

After some crunching of the list in Excel, the results make for some interesting reading.  

Here are the "scores on the doors":
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

| Vendor | Number of Devices without ESXi 7.0 Support |
|:------:|:----------------------------:|
| Adaptec Inc | 72 |
| Advanced Micro Devices	| 8 |
| American Megatrends Inc. | 3 |
| Broadcom | 80 |
| Broadcom / ServerWorks | 13 |
| Compaq Computer Corp. | 9 |
| Dell Inc. | 13 |
| Digital Equipment Corporation	| 5 |
| Emulex Corporation | 124 |
| Hewlett-Packard | 11 |
| HighPoint Technologies, Inc. | 5 |
| Intel Corporation | 153 |
| International Business Machines Corp. | 2 |
| LSI Logic | 83 |
| Mellanox Technology | 25 |
| Neterion Inc. | 2 |
| NetXen Incorporated | 17 |
| NVIDIA | 58 |
| Promise Technology | 24 |
| QLogic Corporation | 15 |
| Silicon Image, Inc. | 12 |
| VIA Technologies, Inc. | 1 |
| **Grand Total** | **735** |

<br>
I've pushed a copy of the spreadsheet into Google Sheets for your full perusal, along with links to the relevant VMware Compatibility Guide (VCG) entries:

<a href="https://docs.google.com/spreadsheets/d/1uWjL0zVi9vQDhhxRo6uWAKjItQY1dkKVmvR0YA7Ocbc/edit?usp=sharing" title="ESXi 7.0 Unsupported Hardware Google Sheet"><img style="display: block; margin-left: auto; margin-right: auto;" alt="Google Sheet" src="/images/esxi-7-the-unsupported/esxi-7-the-unsupported-01.png"></a>

Have a look to see whether your storage controller / network interface / host bus adapter devices are included in the list and hence currently unsupported under ESXi 7.0.

After all, it is often easier to see a complete list in a spreadsheet (Control+F or Command+F to search :wink:) than looking up individual devices in the VCG. 

For completeness, I've posted a copy of the original Dell \UPGRADE\PRECHECK.PY file (renamed to precheck.txt) [HERE](https://polarclouds.co.uk/documents/precheck.zip). See Lines 240 to 1006 of the precheck file for the original list along with the comment *"Devices that are deprecated because of VMKLinux removal"*.

-Chris