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
We proudly announce this LIVE Online Training focusing on Windows Defender Application Control using ConfigMgr and Intune.  It is well-known that Windows Defender Application Control is the security control with the highest effectiveness in stopping malware. Kim Oppalfens and Tom Degreef present this Masterclass.

Fighting malware has become an uphill battle in the last couple of years. This class intends to give you a way to drastically turn the tables by implementing allowlisting based on Microsoft's most recent iteration of the application allowlisting feature. There simply is no functionality out there that has the ability to raise your organization's security posture as much as what application allowlisting can achieve. Best of all, the license for it is already included in your Windows license.

In this LIVE Online Training, you will learn to build and operate a Windows Defender Application Control implementation through ConfigMgr or Intune using an approach recommended by the Microsoft docs, in which the policies are changed as little as possible.

# Hands-on Labs #
Over 60 percent of the time you spend in this live training is dedicated to hands-on labs and exercises, all based on  real-life scenarios and realistic methods tested and verified to work in real-world production environments.  The labs are self-paced, and the instructors are highly available for support. The material is built so the lab exercises can be repeated in your environment.

# Added Bonus #
The best part of this LIVE Online Training is that you will receive access to all the sample files and scripts used during training.  With these, you can rerun the exercises after completing the training and modify them for use in your environment.


# This Live Online Training runs for five days and includes: # 

- Two four-hour live webinars a week, handling two modules a day (with plenty of time for Q&A)
- A community of individuals with the same goals
- A private Facebook group with other participants for sharing reflections, progress, etc.
- The chance to ask Kim and Tom, two seasoned WDAC professionals, questions directly in a live training environment

# This LIVE Online Training is for YOU if you want to: #

- Learn how to implement the single most effective security control against ransomware and other malware
- Master Windows Defender Application Control
- Effectively implement WDAC using ConfigMgr
- Effectively implement WDAC using Intune
- Achieve control and troubleshoot Windows Defender Application Control
...and so much more!

# Prerequisites #
Basic understanding of networking fundamentals such as TCPIP and DNS. Knowledge of Active Directory and Configuration Manager. Scripting experience (VBScript, PowerShell) is helpful but not a requirement.

# Date and Time for Live Webinars #

Dates and start times for the next 5-Day Masterclass:

- Tuesday, April 16, 2024, 9:00 AM-12:30 PM Central Time (US and Canada)
- Thursday, April 18, 2024, 9:00 AM-12:30 PM Central Time (US and Canada)
- Tuesday, April 23, 2024, 9:00 AM-12:30 PM Central Time (US and Canada)
- Thursday, April 25, 2024, 9:00 AM-12:30 PM Central Time (US and Canada)
- Tuesday, April 30, 2024, 9:00 AM-12:30 PM Central time (US and Canada)

[REGISTER HERE](https://academy.viamonstra.com/order?ct=5a7fcf5d-8ed5-4b6a-b6c3-edeb94d680df)

# Masterclass Outline # 
## Module 1: Introduction ##

* Microsoft's Application Allowlisting History
* Windows Defender Application Control and AppLocker Comparison
* Measuring Application Allowlisting Effectiveness
* High-Profile Threat Campaigns vs Application Allowlisting


## Module 2: PowerShell Constrained Language Mode ##

* The PowerShell Execution Policy Explained
* Overview of the Different PowerShell Language Modes
* PowerShell's Dot Sourcing Feature and WDAC
* Codesigning PowerShell Code

## Module 3: WDAC Basics - Rule Options ##

* The Base and Supplemental Policy Format
* Analyzing the Available Policy Rule Options
* Inspecting the Windows Default Policies
* WDAC Tooling: The WDAC Wizard for Building and Editing Policies
* The Intelligent Security Graph's Role

## Module 4: WDAC Basics - CI Rules ##

* Rule Types: Allow and Deny
* Certificate-based Rules
* File Object Characteristics-based Rules
* Signing Scenarios: Windows and Driver Mode
* Path Rule Gotcha's
* Handling Packaged Apps (aka Modern or Store Apps)
* Rule Processing Explained

## Module 5: WDAC Managed Installers ##

* The Magic of Managed Installers
* Creating Your Own Managed Installers
* Understanding the Impact of Process Trees
* NTFS Extended Attributes in WDAC
* Managed Installer Logging
* Known Issues

## Module 6: Windows Security Catalog Usage and WDAC ##

* Security Catalog Basics
* Creating Security Catalogs
* WDAC Tooling: Package Inspector
* Automating Package Inspector
* Catalog Management
* MSIX as a Catalog


## Module 7: ConfigMgr and Intune Policies for WDAC ##

* ConfigMgr Policy Options
* WDAC and .NET Native Images
* Things Learned from Reverse Engineering the ConfigMgr Implementation
* Impact of the Operating System Default Policies on the Resulting ConfigMgr Policy
* Managed Installer Applied to Microsoft Systems Management Solutions
* Intune Policy Options
* Things Learned from Reverse Engineering the Intune Implementation
* Intune Caveats

## Module 8: WDAC and Server Operating Systems ##

* Selecting WDAC Prime Candidate Server Roles
* Supported WDAC Features per Operating System Server SKU
* Building (Server) Application Policies
* Building a Domain Controller Policy
* Building a Certificate Server Policy
* Building a SQL Server Policy
* Azure Plugins and Agents


## Module 9: WDAC Logging ##

* Understanding the Events Logs
* Understanding the Application Allowlisting Event IDs
* The Event ID Tags
* To Be Signed (TBS) Hashes and WDAC
* The Correlation ID in the Event Log
* Options to Centralize the Event Log

## Module 10: Developing an Implementation Plan ##

* Making Your Application Allowlisting Case
* Lifecycle of a Ransomware Incident
* Kick Off Your Implementation
* Authoring a Communication Strategy
* Handling the End User Impact of Interactively Launched Processes
* The Microsoft Recommended Block Rules
* Working with Multiple WDAC Policies