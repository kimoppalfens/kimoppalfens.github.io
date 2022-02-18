---
title: "The best of MEM, must read fridays - wk2206 ."
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2022-02-11
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
Some promo for a post of my own! When I originally tweeted out our toast-notification for users that had an interactive process blocked the tweet received over a 100 likes and severeal retweets alongside a number of replies on how this was achieved. This blogpost details the why and the how. [Modern end-user communication on a WDAC enforced device.](https://www.oscc.be/mem/wdac/Modern-end-user-communication-on-a-WDAC-enforced-device/)

## Item 2 ##
This one is a bit special, not a blogpost but just a tweet. Nevertheless it is life-altering-goodness. Thanks [Jóhannes](https://twitter.com/jgkps)  for sharing this. [TIL: you can get auto completion in @code using https://marketplace.visualstudio.com/items?itemName=eliostruyf.vscode-msgraph-autocomplete… for graph. This makes it so much easier to code for #intune](https://twitter.com/jgkps/status/1490013927941492741)

## Item 3 ##
A security issue was uncovered where the msix protocol handler could be used to spoof the publisher of the msix. The issue was being used by malicious adversaries and as a result the handler was disabled by Microsoft. This prevents auto-installs using the string ms-appinstaller:
It doesn't prevent you from distributing msix's with #Intune or #SCCM though, nor does it prevent you from downloading the msix and installing it that way. This showed up in my twitter feed courtesy of [Garrett](https://twitter.com/gwchandler).
More details:
[Disabling the MSIX ms-appinstaller protocol handler](https://twitter.com/gwchandler/status/1489738026162868226)
[TechCommunity Article](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/disabling-the-msix-ms-appinstaller-protocol-handler/ba-p/3119479)
[Sophos labs sharing their find with technical details on how this works.](https://news.sophos.com/en-us/2021/11/11/bazarloader-call-me-back-attack-abuses-windows-10-apps-mechanism/)


## Item 4 ##
This one rocked the Infosec community and appeared multiple times in my feed. A win for the defense team. Microsoft, finally, decided that enough was enough with Microsoft apps for enterprise / Office Macro's. Office Macro's in documents delivered from the internet will now be blocked by default. I'd still like to see an easy way to identify  users / devices that haven't used macro's within a given period so I can disable macro's alltogether on these devices. But I, for one, definitely appreciate this [Helping users stay safe: Blocking internet macros by default in Office](https://techcommunity.microsoft.com/t5/microsoft-365-blog/helping-users-stay-safe-blocking-internet-macros-by-default-in/ba-p/3071805?utm_source=dlvr.it&utm_medium=twitter)

## Item 5 ##
I always have a keen interest in Cloud Management Gateway related items. Plenty of organizations would've been in quite a bit of additional hurt without a CMG. The cloud heavy ConfigMgr is a great title and a great concept. A cloud-heavy CMG is a wonderful additional to managing your Windows devices or wonderful foundation where #Intune is the wonderful addition. I appreciate the insight into how Microsoft uses CMG's as I've been impressed by what the technology brings to the table from day 1. [The cloud heavy ConfigMgr - Part 2](https://techcommunity.microsoft.com/t5/device-management-in-microsoft/vmss-based-cmgs-and-the-cloud-heavy-configmgr-part-2/ba-p/3095255)


## Item 6 ##
I'll close out with an Autopilot post. Written by [mr_helaas ](https://twitter.com/mr_helaas), the joke in the name doesn't really translate to English, but I definitely laughed. An extensive post on how to [handle computernames in Autopilot scenarios.](https://endpointcave.com/set-up-the-missing-naming-convention-to-increment-your-autopilot-devices/) 







