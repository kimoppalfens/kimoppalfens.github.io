---
title: "MVP Summit hackathon edition 2016-Part 3"
author: Kim Oppalfens
related: true
date: 2016-12-12
categories:
  - SCCM
  - Configmgr
tags:
  - Hackathon
---

This years MVP summit took place a little bit over a month ago, and I am feeling like I am in a permanent state of jet-lag.

# Intro #
A fun and amazing part of that conference, that was introduced last year, and repeated this year, is the Hackathon. You can read about the first part of my reporting on this amazing event here: [http://www.oscc.be/sccm/configmgr/MVP-Summit-Hackathon-edition-2016-Part-1/](http://www.oscc.be/sccm/configmgr/MVP-Summit-Hackathon-edition-2016-Part-1/)

This year's edition had no less than 13 different little hacks. So let's start by introducing some more.

## Force compliant check ##
I've lost count of how many hackathon projects I've already covered, but next one on the list is the ability to quickly force down a compliant baseline, and get the results back in a swift manner. Main goal of this project, is obviously to do this without needing additional communication requirements like firewalls rules, permissions etc... The perfect manner of handling this would be through the Big Green Button aka Fast notification Channel.

**Related uservoice items**: 
[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8374809-include-the-right-click-tools-kit-as-part-of-the-c](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8374809-include-the-right-click-tools-kit-as-part-of-the-c) (580 votes)

## Monitoring automation aka Better SCCM + OMS Integration ##
This team's intro was SCCM + OMS Makes dreams comes true. I guess this one is for those folks out there that dream about OMS. The idea is to make a lot of data that CM has actionable in OMS, and allow for the correlation of SCCM data with all the other data that OMS can consume.

The current thinking is to use Azure datacollector Rest API's to have a JSON uploader send data to the OMS analytics service. The demo wasn't as sexy as using cortana used in the MSGraph hackathon, but the whole idea of having the data in OMS and allowing one to create actionable alerts using webhooks or runbook does show a lot of promise. One example where this could be useful is to action and re-mediate compliance based on the data collected and correlated.

## The special team - Patch at Shutdown  ##
This team was mainly tasked with getting the 1610 build out of the door for us to install when we arrived back at our offices after an exhausting MVP Summit. Because of this, they didn't manage to actually write a ton of code, and largely stuck to PowerPoint.

What was clear is that this brainstorming was done by technical folks, and the marketing team hadn't been involved yet. This became apparent by the current "technical name" of the feature. Some features get a technical name, that is later reworked by the marketing department. A recent well-known example is the Big Green Button that was given the "Fast notification channel name" Or Bits-enabled distribution points in a less recent past that were originally called Drizzle points. This team called their feature "Patch when I am heading home". 

The team did demo the things that they did work on for the 1610 build, which is a crucially important part in Windows 10 management through SCCM going forward, the support for Express updates.

PS: I thought it was rather interesting that this item had the Number of the beast amount of votes at the time of writing this blog.


**Related uservoice items**: 
[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8792341-install-and-shutdown-install-and-reboot-with-sof](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8792341-install-and-shutdown-install-and-reboot-with-sof)(666 votes)


## The end score ##

### First 3 features by Number of votes ###


1.  **The tasksequence debugger** aka *genesis* team

2. Client Analytics Dashboard
3. Microsoft Graph - Coming to a CM Admin UI soon(ish)

### Feature that received the largest number of "*Ship first votes *"

2 teams ex-aequo'ed for the largest number of Ship first votes:
1. The implicit uninstall team
2. The MS Graph team

So, my team eventually came in 3rd, but from the looks of it we'll all be winners if this podium makes it into a build in the (very) near future. The *genesis* team can call themselves **2nd MVP Hackathon Summit Grand Champion** for the rest of the year. 




