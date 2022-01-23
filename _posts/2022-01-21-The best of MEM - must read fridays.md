---
title: "The best of MEM must read fridays - wk2203 ."
header:
  image: Wdac-Tom-1280-x960.jpg
  teaser: Wdac-Tom-512-x384.jpg
author: Kim Oppalfens
date: 2021-01-21
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

[Intune grouping, targeting, and filtering: recommendations for best performance](https://techcommunity.microsoft.com/t5/intune-customer-success/intune-grouping-targeting-and-filtering-recommendations-for-best/ba-p/2983058)

This blog post by Scott offers some insight and best practices into using groups for targetting in in Intune. The new filter capabilities offer some interesting options in this respect. The post offers Do's and Dont's on policy targetting. 

## Item 2 ##
[Securing Intune Enhanced Inventory with Azure Function](https://msendpointmgr.com/2022/01/17/securing-intune-enhanced-inventory-with-azure-function/)
The folks over at msendpointmgr already had an interesting set of blogs on collecting custom items from clients and store them in a log analytics repository for inventory and reporting purposes. The original solution shared out the workspace id and sharedkey which certain organizations could find overpermissive. This new approach uses an intermediate Azure function to abstract the need for the id and key. The Azure function verifies the input and only acts on validated input. That's an interesting concept, that, just like the original post, can be used for other purposes where you want to store client data centraly. 

## Item 3 ##
[Expert Guide to Microsoft January 2022 OOB Patches with WSUS and Configuration Manager](https://skatterbrainz.wordpress.com/2022/01/19/expert-guide-to-microsoft-january-2022-oob-patches-with-wsus-and-configuration-manager/)

A skatterbrainz post on the issues with the january updates and how the handle them with wsus and ConfigMgr. The post contains the step-by-step instructions on how to handle the re-released updates to rectify the january update cycle issues.

## Item 4 ##
[Monitoring Windows Autopilot deployments with Azure Logic Apps and Adaptive Cards for Teams](https://www.petervanderwoude.nl/post/monitoring-windows-autopilot-deployments-with-azure-logic-apps-and-adaptive-cards-for-teams/)
I picked this post because of the sheer number of new'ish parts it uses in a solution. Autopilot, Azure KeyVault, Microsoft graph, Azure logic apps and teams cards, what's not to like. Peter turned this into an extensive post and I look forward to playing with the automated teams cards in automation myself.

## Item 5 ##
[Why you shouldn’t set these 25 Windows policies](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/why-you-shouldn-t-set-these-25-windows-policies/ba-p/3066178)
Another extensive blogpost full of actionable advices around Windows Update setttings for both group policy and Configuration Service provider settings. This post would've been hard to pass on in any particular circumstances. The author, Aria recommends it as the best post she's ever written.  




