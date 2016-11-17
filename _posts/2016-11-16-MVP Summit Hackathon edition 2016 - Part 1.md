---
title: "MVP Summit hackathon edition 2016 - Part 1"
author: OSCC
date: 2016-11-16
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

This years MVP summit took place last week, and I am still recovering from jet-lag.

# Intro #
A fun and amazing part of that conference, that was introduced last year, and repeated this year, is the Hackathon.

When the concept was first introduced to us, most of us, if not all had never heard of a Hackathon. In short, it's a development exercise, where a certain feature is whipped up in a short time frame. A more extensive explanation can be found on Wikipedia:
[https://en.wikipedia.org/wiki/Hackathon](https://en.wikipedia.org/wiki/Hackathon)

This year's edition had no less than 14 different little hacks. So let's start by introducing them one by one.

## Client Analytic's dashboard / Donuts ##
Team that had the largest MVP count to engineer ration. This hackathon is about making a ton of additional information about a machine available in a client analytic's dashboard. Could show the following information (Example and nothing guaranteed)
- Pending Reboots
- SwUpdate deployment status
- Other deployment statuses

Could be made available for mobile devices as well

**Related uservoice items**: [https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/14743734-create-missing-updates-overview-and-detail-repor](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/14743734-create-missing-updates-overview-and-detail-repor) (12 votes)

[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8468233-reboot-pending-reporting](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8468233-reboot-pending-reporting) (12 votes)


## Implicit uninstall ##
This team was coding till the very last minute of their demo. The functionality they bring to the table is uninstalls being triggered when a deployment is no longer targeted at a given device. As a bonus they included the ability to uninstall when a software has not been used in X amount of time (Requires the SCCM Admin to specify which process executable are considered the application as used.) For apps that need approval, the previously approved request could be dynamically deleted.

**Related uservoice items**:[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8344038-option-to-uninstall-an-application-when-a-user-or](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8344038-option-to-uninstall-an-application-when-a-user-or) (313 votes)



## Scripts node and reviewer RBA role ##
I was partially involved in this project where you get two for the price of one project. This project want to add a Scripts node, or easy scripting deployments to the Admin UI. The idea is to allow the easy execution of approved scripts to devices. Idea is that scripts don't have to go through distribution points, and can be quickly delivered to devices through the fast notification channel. This scripts node could equally host scripts that are reusable in locations in the admin UI where scripts can be used (Detection Methods, CI')

As this is powerful, and as a result risky from a feature perspective this is coupled with a new Reviewer RBA role to have the scripts approved before sending. This could optionally require 2-key approval, where an author can't approve his own scripts. To be workable this project needs quite a bit of architectural changes which might mean we'll have to wait for this one a bit longer

## MS Graph access to the SMS Provider  ##
This was the team I spend most of my time with. Microsoft Graph is a unified API endpoint, for accessing data, intelligence, and insights coming from the Microsoft cloud or on-premise graph enabled applications. This might not mean much to many IT Pro's but this projects ties a number of cloud services together alongside with all the data we have available in SCCM. I am incredibly excited about this as it would open up access to and from:
- Power BI
- The Internet
- Azure
- Intune
- Office 365
- Azure AD
- Mobile OS'es
- IFTTT
- Microsoft Flow
- Cortana
- ...

The possibilities with this seem to be endless, ranging from requesting / manipulating data using Cortana (Fancy, but not the must usable), over accessing your data in an automatically generated Power BI model. Followed by Multi factor authentication for an Admin UI, or even a mobile device compatible/specific UI. With all of the internet access protected by the new Cloud Management gateway functionality.

**Related uservoice items:**
[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8558383-improved-automation-and-extensibility-by-exposing](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/8558383-improved-automation-and-extensibility-by-exposing) (62 votes)

[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/9424425-integrate-configuration-manager-reporting-with-pow](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/9424425-integrate-configuration-manager-reporting-with-pow) (19 votes)


 