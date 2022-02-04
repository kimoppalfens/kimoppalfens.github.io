---
title: "The best of MEM, must read fridays - wk2204 ."
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2022-01-28
categories:
  - MEM
tags:
  - SCCM
  - ConfigMgr
  - Intune
  - MEM
---

This post highlights the most interesting blogs, tweets and articles I've read relating to MEM in the past week.

# Intro #

I figured I'd try and start something new. To stay up-to-date on what's happening in the MEM and Windows management realm I read a ton of stuff. A serious amount of community effort and a similar amount of items Microsoft puts out. The amount of things being put out is all incredibly valuable, and covers all types of knowledge depth imaginable. On a weekly basis I am going to try and shine a light on some of those items that I personally believe you must read. I'll equally add my view as to why I believe this is something you should probably look into. Items appear in random order.

## Item 1 ##
[Achieve better patch compliance with Update Connectivity data](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/achieve-better-patch-compliance-with-update-connectivity-data/ba-p/3073356)

This blog kicks in an open door a bit. If you're not connected and you centrally manage your patch deployment than your device isn't going to get updated, duh. Most of us already knew that bit. This post however introduces you to a metric Microsoft introduced in measuring connectivity cleverly named "**Update Connectivity**" The blog continues on with how that metric is being used in Intune reports. Community members or me, whomever figures it out first, might want to start looking whether we can find this data on a device directly.


## Item 2 ##
[Always On VPN Windows 11 Issues with Intune](https://directaccess.richardhicks.com/2021/10/28/always-on-vpn-windows-11-issues-with-intune/)
I like to include items that are actionable. This one by Richard unfortunately isn't. What it does do is save you a ton of time in trying to remediate an issue with Windows 11 Intune deployed Always on VPN profiles dis/re-appearing. If you see this issue, you're not alone and if Richard Hicks states something around Always on VPN like, "*There's currently no workaround*" you can take that as, let's not bother and wait for a fix. Not necessarily actionable but allows you to focus your time elsewhere. Thanks Richard. 

## Item 3 ##
[https://www.petervanderwoude.nl/post/getting-started-with-filtering-and-selecting-microsoft-intune-data-via-microsoft-graph/](https://www.petervanderwoude.nl/post/getting-started-with-filtering-and-selecting-microsoft-intune-data-via-microsoft-graph/)
Peter Vanderwoude typically puts out at least one blog a week and most of them tend to trigger me into reading them. This one's no different. Automating things with Microsoft Graph is a somewhat must-have skill now-a-days in the MEM world. Peter covers filtering and the basic / most common operators to use in filtering Graph data here.

## Item 4 ##
[SSO to domain resources from Azure AD Joined Devices – The MEGA Series – Part 3 – Configure the VPN Server](https://msendpointmgr.com/2022/01/22/sso-to-domain-resources-from-azure-ad-joined-devices-the-mega-series-part-3-configure-the-vpn-server/)

Ben offered me a way to sneak his blog on SSO / Kerberos auth for Azure AD only joined devices in this weeks mustreads. Technically the Part1 of the series was made available several moons ago. A new part in the series came online this week though, (I know, technically last week, but I read that part this week.). So there you have it, part 1 is a good place to start if you haven't read it and if you have, it's probably worth a revisit.





