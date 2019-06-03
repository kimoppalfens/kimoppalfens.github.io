---
title: "Windows Defender Application Control"
author: OSCC
header:
  overlay_image: updatesandservicing1280x960.jpg
  teaser: updatesandservicing512x384.jpg
date: 2001-06-03
categories:
  - osccservices
category: osccservices
excerpt: "Let us help you implementing Application whitelisting (DeviceGuard)"
tags:
  - SCCM
  - vulnerability
  - security
  - Windows 10
---

# What is device guard? #

Device guard is the original name Microsoft used to introduce its next generation application whitelisting solution. The technology and announcement coincided with the release of Windows 10 Enterprise. The feature is sometimes referred to as Windows defender application control or configurable code integrity.

With thousands of new malicious files created every day, using traditional methods like antivirus solutions—signature-based detection to fight against malware—provides an inadequate defense against new attacks. In most organizations, information is the most valuable asset, and ensuring that only approved users have access to that information is imperative.

However, when a user runs a process, that process has the same level of access to data that the user has. As a result, sensitive information could easily be deleted or transmitted out of the organization if a user knowingly or unknowingly runs malicious software.

Application control is a crucial line of defense for protecting enterprises given today’s threat landscape, and it has an inherent advantage over traditional antivirus solutions. Specifically, application control moves away from the traditional application trust model where all applications are assumed trustworthy by default to one where applications must earn trust in order to run.

# Isn’t application whitelisting extremely hard? #

An often-repeated remark around application whitelisting is indeed that implementing and maintaining the whitelist is a challenge. In an environment where most of your software is deployed using a systems management tool like System Center Configuration Manager it doesn’t necessarily have to be. Configuration Manager Current Branch has introduced the managed installer allowing for a convenient way to deploy trusted applications.

# Why application whitelisting? #

Application whitelisting reduces the code allowed to run on a system like tablets and smartphones. When these types of devices can run code from anywhere, by adding unauthorized stores, jailbraiking or rooting them, they are considered a security risk. They are then dealt with accordingly by IT and security departments. Strangely enough, a Windows system that can run code from anywhere isn’t treated in the same fashion. Preventing random code to run on a system increases its security posture by an order of magnitude. Limiting the code bad actors can use on a system heavily reduces their effectiveness.  And increases the effort they must spend to perform their malicious actions. 

The result of this is like securing your house properly in the physical world. If you do, chances are the bad actors will move on to the next house and leave yours alone.

# Where can OSCC assist? #

## Custom Application Whitelisting training ##

Given the focus of OSCC on building secure desktop management services OSCC can assist in several ways. A custom Microsoft application whitelisting training was developed to get organizations with an interest in this solution up to speed. This training has been very well received by customers that have sat in on it.

The training delivers the technical details needed to get going but equally contains an implementation approach including ideas around end user communication.

## Proof of concept consultancy ##

OSCC has built an approach to Windows Defender Application Control that goes hand in hand with a systems management solution like SCCM.

This approach starts from a strict policy that mainly trusts just the Operating System binaries. The policy is subsequently enhanced with trusts for the applications you have already deployed using SCCM. Applications previously deployed are unfortunately not covered by SCCM’s managed installer. These applications are typically the biggest hurdle in moving to an application whitelist enforced environment. 

To assist with this OSCC has developed a Windows Defender Application trust engine that can take a list of applications in XML format and generate a trusted catalog out of that list. Additionally, an SCCM admin UI extension was developed to assist in generating this XML file. The result of these 2 items combined gives customers the ability to build an extensive, tailored policy that trusts the organizations current applications rapidly but without creating a policy that is too tolerant risking easy bypasses. 

The other challenge in implementing application whitelisting is knowing when your policy is ready for enforcement on devices. OSCC’s centralized inventory, which integrates into SCCM or other systems management solutions. This provides customers with a centralized view of blocked applications to help them decide when the time for enforcing the policy has arrived.

# How much does the Microsoft application whitelisting training cost? #
The training is priced at 9.500€ for up to 8 participants.

# How much does the proof of concept consultancy cost? #
The consultancy is billed per consultancy day and depends on the agreed scope of the poc.
