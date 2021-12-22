---
title: "802.1X is not working properly with Windows 11 ADK"
header:
author: Tom Degreef
date: 2021-12-22
categories:
  - Windows

tags:
  - SCCM
  - Configuration Manager
  - Endpoint Manager
  - OSD
  - ADK
---

Dot3Svc won't start in default Windows 11 ADK

## Intro ##

If you use NAC (Network Access Control) during OSD, you might run into an issue when you upgrade to the Windows 11 ADK (Version 10.1.22000.1)

This short blogpost should help you get around that issue.

## Problem 1 ##

802.1X requires the WinPE-Dot3svc package to be added to your WinPe image. The problem is, even if you add the package, the Dot3svc-service doesn't start in Windows PE. This is due to the fact that  Microsoft forgot to include a DLL file (mobilenetworking.dll) when creating the WinPE-Dot3Svc package.

To work around this issue, you can copy the mobilenetworking.dll into the System32 folder in WinPE (depending on how you use WinPE, this might require you to manually mount the image and copy the file by hand).

This mobilenetworking.dll file can be found in a regular/full windows 11 pro/enterprise version. Just gather the file from there.

## Problem 2 ##

Ok, so with the workaround from Problem 1, we can now start the dot3svc service ! But you might still have issues with 802.1x authentication.

To fix that particular issue, you have to add some registry keys into your WinPE image :

- Key : HKLM\WinPE\ControlSet001\Services\EAPHost 
- Name : UseLegacyTlsStack
- Type : REG_DWORD
- Value : 1

This should probably also be done by mounting the WinPE Wim file, then loading the System hive of your WinPE by running :
**reg.exe load HKLM\WinPE Mount Path\Windows\System32\Config\SYSTEM**

Once the correct hive is loaded, you can add the required registry value like this :
**reg add HKLM\WinPE\ControlSet001\Services\EAPHost /t REG_DWORD /v UseLegacyTlsStack /d 1**

Finally, you need to unload the system hive by running :
**reg.exe unload HKLM\WinPE Mount Path**

Unmount the WIM and commit the changes. This should fix the issue with the 802.1x authentication.

Good luck getting NAC up & running with the latest Windows 11 ADK!

Take care,

Tom
