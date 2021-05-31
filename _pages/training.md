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
OSCC will organise a new training focussed on one of the best Windows 10 security features. Application control (or application whitelisting) allows you to increase your security posture many times since only approved applications are allowed to run. 

This training will take you through all the necessary topics in order to be able to properly implement an application control policy in your environment.  We will focus on protecting core servers and workstations in your environment. 

# Practical details #
Each module will be presented live and will run for about 2 hours. Trainings will run twice a week (Tuesdays and Thursdays) at 7PM CET (10AM PST / 1PM EST) for about 5 weeks.

students will get access to a personal online lab environment to run exercices that are specifically crafted for this training. Once the training is over, students will be able to download the lab offline for continued access.

A discord channel will be available for questions and follow up

# When #
The training will start on Tuesday 9th of November 2021 and will finish Tuesday 7th of December 2021.

[Register here](https://www.eventbrite.com/)

# Modules #
## Intro ##

Applocker, Windows Defender Application control's little brother, is the predecessor application approval technology Microsoft developed. An understanding of their differences similarities and how they interact is hugely beneficial to understand Application Control in general.

## PowerShell Constrained Language Mode ##

The idea behind controlling what code can run is to increase the barrier of entry for ransomware attacks. If PowerShell is left wide open that goal won't be met. In PowerShell Constrained Language mode we'll look at how to block PowerShell usage from the bad folks out there but still keep it available for your own Systems Management needs.

## WDAC Basics - rule options ##

This is where we start looking into the elements of a WDAC policy. The highest level items in a WDAC policy are the rule options. These impact the overall behavior of the policy. In this module you'll get an introduction to what the rule options do and how to build your first policy.

## WDAC Basic - CI rules ##

Code Integrity (CI) Rules are objects to allows certain code/software to run on your systems. There are multiple ways of making software trusted, CI Rules is one of those ways. CI Rules come in different shapes and forms. This module is about explaining what sort of rules are available to you and how they work.

## WDAC Managed Installers ##

In the previous module we saw one way of making applications/code trusted. Building rules for every piece of software can be tedious. WDAC Managed Installer functionality is a flexible way to make applications/code trusted in an enterprise environment that relies on a Microsoft systems management solution. After this part you'll understand the MI functionality and its challenges.

## MEMCM / Intune policies ##

With a thorough understanding of the mechanics of Managed Installers, let's move on to actually implementing this functionality using Microsoft's systems management solutions. This module will look into the options available with both Microsoft Endpoint Manager Configuration Manager and MEM ( Intune).

## WDAC for Servers ##

Servers typically have a more static workload than workstations have. This makes them ideal candidates for an application control solution. Not all servers are created equal though. This section will focus on helping you build policies for 2 types of servers that, when compromised, can mean the loss of your entire environment. We're talking about Domain Controllers and System Center Manager Configuration Manager servers. Both have the capabilities to distribute code to a large set of devices within your organization to wreak havoc.

## Centralized Logs (ATP, Windows Event Forwarding, SCCM) ##

WDAC logs events locally in the Windows Event viewer. To operationalize a WDAC practice a way to centralize these logs is indispensable. In this module we'll look at 3 different options to achieve this centralization.

## Developing a WDAC Implementation plan ##

Actually building a WDAC practice is what this training is all about. In the final section of this training we'll look at everything you've learned so far and how you combine this into a manageable solution that drastically increases your security posture.