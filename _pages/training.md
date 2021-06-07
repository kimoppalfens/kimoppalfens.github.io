---
title: Windows Defender Application Control
author: Win10 Training 
layout: single
excerpt: "training overview"
sitemap: false
permalink: /training/
author_profile: true
---

# Overview #
OSCC is organising a new training focussed on one of the most powerful Windows 10 security features. Windows Defender Application control is an application allow-listing solution that allows you to take your security posture to a whole new level. It does so by controlling which applications are allowed to run and helps you limit the sources where code can come from to a limited set of items you manage. 

This training will provide you with all the necessary pieces of information, caveats and insight knowledge gained through our vast experience in this topic in order to be able to properly implement an application control policy in your environment.  The focus of this training is the protection of your core servers (AD, PKI, MEMCM) and workstations in your environment. 

# Practical details #
Each module will be presented live and will run for about 2 hours. Trainings will run twice a week (Tuesdays and Thursdays) at 7PM CET (10AM PST / 1PM EST) for 5 weeks.

students will get access to a personal online lab environment to run exercices that are specifically crafted for this training. Once the training is over, students will be able to download the lab offline for continued access.

A collaboration channel will be available during and after this training, for questions, follow-up and sharing experiences amongst people that are working on a similar project.

# When #
The training will start on Tuesday 12th of October 2021 and will finish on Thursday the 25th of November 2021.
There won't be any sessions over the MMS Miami break end of October till early november.

[Register here](https://www.eventbrite.be/e/windows-defender-application-control-training-tickets-157760422671?keep_tld=1)

# Module details #
## Intro ##

Applocker, Windows Defender Application control's little brother, is the predecessor application approval technology Microsoft developed. An understanding of their differences, their  similarities and how they interact is hugely beneficial to help you understand Application Control in general.

## PowerShell Constrained Language Mode ##

The idea behind controlling what code can run is to increase the barrier of entry for ransomware attacks. If PowerShell is left wide open that goal won't be met. In PowerShell Constrained Language mode we'll look at how to block PowerShell usage from the bad folks out there but still keep it available for your own Systems Management needs.

## WDAC Basics - rule options ##

This is where we start looking into the elements of a WDAC policy. The highest level items in a WDAC policy are the rule options. These impact the overall behavior of the policy. In this module you'll get an introduction to what the rule options do and how to build your first policy.

## WDAC Basic - CI rules ##

Code Integrity (CI) Rules are objects to allows certain code/software to run on your systems. There are multiple ways of making software trusted, CI Rules is one of those ways. CI Rules come in different shapes and forms. This module is about explaining what sort of rules are available to you and how they work.

## WDAC Managed Installers ##

In the previous module we saw one way of making applications/code trusted. Building rules for every piece of software can be tedious. WDAC Managed Installer functionality is a flexible way to make applications/code trusted in an enterprise environment that relies on a Microsoft systems management solution. After this part you'll understand the MI functionality and its challenges.

## Working with Catalogs ##
Using security catalogs is a way to make files trusted that is being used by drivers for over 2 decades. This proven technology can equally be used to make application files trusted for Applocker. This section will cover managing .cat files and the newer msix format as a way to make applications trusted.

## MEMCM / Intune policies ##

With a thorough understanding of the mechanics of Managed Installers, let's move on to actually implementing this functionality using Microsoft's systems management solutions. This module will look into the options available with both Microsoft Endpoint Manager Configuration Manager and MEM ( Intune).

## WDAC for Servers ##

Servers typically have a more static workload than workstations have. This makes them ideal candidates for an application control solution. Not all servers are created equal though. This section will focus on helping you build policies for 2 types of servers that, when compromised, can mean the loss of your entire environment. We're talking about Domain Controllers and System Center Manager Configuration Manager servers. Both have the capabilities to distribute code to a large set of devices within your organization to wreak havoc.

## Centralized Logs (ATP, Windows Event Forwarding, SCCM) ##

WDAC logs events locally in the Windows Event viewer. To operationalize a WDAC practice a way to centralize these logs is indispensable. In this module we'll look at 3 different options to achieve this centralization.

## Developing a WDAC Implementation plan ##

Actually building a WDAC practice is what this training is all about. In the final section of this training we'll look at everything you've learned so far and how you combine this into a manageable solution that drastically increases your security posture.