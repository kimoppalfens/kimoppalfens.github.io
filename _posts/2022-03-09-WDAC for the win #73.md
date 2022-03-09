---
title: "WDAC for the win #73 - The one with Nvidia"
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2022-03-09
categories:
  - WDAC
tags:
  - WDAC
  - ConfigMgr
  - Intune
  - MEM
---

This post details howto implement a Wdac policy to block the stolen Nvidia certs.

# Intro #

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">WDAC policies work on both 10-11 with no hardware requirements down to the home SKU despite some FUD misinformation i have seen so it should be your first choice.  Create a policy with the Wizard and then add a deny rule or allow specific versions of Nvidia if you needpic.twitter.com/gcNLJoJ0Gn</a></p>&mdash; David Weston (DWIZZZLE) (@dwizzzleMSFT) <a href="https://twitter.com/dwizzzleMSFT/status/1499527802382471188">March 4, 2022</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Item 1 ##

Using pro-active remediations, the Configuration Items from the Intune world, to detect, communicate and remove Unsupported applications. An interesting usecase for Proactive Remediation by [@jankeskanke](https://twitter.com/jankeskanke) and [@byteben](https://twitter.com/byteben). Interesting as a solution but probably even more so to demonstrate how much management overhead and security risks you introduce by allowing end-users to install apps by themselves. [Use Proactive Remediations to pop a Toast Notification when Unsupported Apps are found](https://msendpointmgr.com/2022/02/27/use-proactive-remediations-to-pop-a-toast-notification-when-unsupported-apps-are-found/)

## Item 2 ##
One of the things a number of admins has to learn is how to do troubleshooting for workloads delivered from Intune. Several admins out there have years of experience in troubleshooting using #ConfigMgr logs. These quite clearly aren't used for Intune related functionalities. This blog has a pretty extensive amount of detail on how installing a win32 app is installed by the Intune Management Extension.

[Intune Win32 App Issues Troubleshooting Client-Side Process Flow](https://www.anoopcnair.com/intune-win32-app-troubleshooting/)

## Item 3 ##
One of the many pieces of the puzzle you'll need to replace your GPO's with Intune settings management. [Manage Power Settings via Microsoft Intune](https://blog.mindcore.dk/2022/02/manage-power-settings-via-microsoft.html)  a blog by
[@MMelkersen](https://twitter.com/MMelkersen)
[@kennethvs](https://twitter.com/kennethvs)
and
[@SandeRozemuller ](https://twitter.com/SandeRozemuller)







