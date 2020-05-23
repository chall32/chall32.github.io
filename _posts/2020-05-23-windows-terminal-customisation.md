---
layout: post
title: "Windows Terminal Customisation" 
excerpt: "Non-Terminal Tweaking"
tags: 
- Free
- Pro-Tip
- Windows
image:
  thumb: windows-terminal-customisation/windows-terminal-customisation-00.png
comments: true
date: 2020-05-23T00:00:00+00:00
---
<img style="float: right; margin: 0px 0px 10px 10px;" alt="Windows Terminal" src="/images/windows-terminal-customisation/windows-terminal-customisation-00.png">
Windows Terminal is the new -er- terminal application from Microsoft.  It features tabbed terminal access to shells like Command Prompt, PowerShell and Windows Subsystem for Linux (WSL).  On top of that, Windows Terminal features Unicode and UTF-8 character support, a GPU accelerated text rendering engine and custom themes, styles, and configurations.

What's more is that it's free and open source, with the source code hosted on GitHub. [Take a look!](https://github.com/microsoft/terminal/)

Install Windows Terminal from the [Microsoft Store](https://aka.ms/terminal)

{% include _toc.html %}

## Customisation
Once installed, You'll want to customise. Here's how.

### Accessing Windows Terminal Settings
Access windows Terminal via the drop down menu option:
 
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Settings" src="/images/windows-terminal-customisation/windows-terminal-customisation-01.png">

This will then open your Windows Terminal user settings in either notepad or (depending on your file association setup) the much preferred [notepad++](https://notepad-plus-plus.org/) as I'm using below. 

### Default Terminal
Out of the box, Windows Terminal opens a PowerShell shell by default. Lets change that back to Command Prompt.

Replace the `"defaultProfile"` GUID with the GUID from Command Prompt:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Command Prompt GUID" src="/images/windows-terminal-customisation/windows-terminal-customisation-02.png">

Save and restart windows terminal to test.

### Starting Folder
To change the Windows Terminal opening folder, add the option `"startingDirectory": "",` to the profile of the terminal you wish to change. 

**NOTE:** Use forward slashes rather than the usual backslashes used by Windows - yeah, "reasons" I guess.

For example, to open PowerShell in C:\Scripts folder:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="PowerShell Start Folder" src="/images/windows-terminal-customisation/windows-terminal-customisation-03.png">
 
Save and restart windows terminal to test.

### Background Image
To add a background image, add the `"backgroundImage": "",` option the the profile of your choice. Image opacity is set using values of `"backgroundImageOpacity":` between 0 and 1:

<img style="display: block; margin-left: auto; margin-right: auto;" alt="Background Image" src="/images/windows-terminal-customisation/windows-terminal-customisation-04.png">

Which in-turn results as:
<figure>
<img style="display: block; margin-left: auto; margin-right: auto;" alt="Cover Me, I'm Going In!" src="/images/windows-terminal-customisation/windows-terminal-customisation-05.png">
<figcaption>Cover me... I'm going in!  :smile:</figcaption>
</figure>
## But Wait, There's More!
There are plenty of other Windows Terminal tweaks possible.

For further details, take a look at [Profile settings in Windows Terminal](https://docs.microsoft.com/en-gb/windows/terminal/customize-settings/profile-settings) on Microsoft Docs for the full run down.

-Chris