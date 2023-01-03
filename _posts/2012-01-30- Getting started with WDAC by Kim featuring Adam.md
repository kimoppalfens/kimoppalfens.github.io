---
title: "Getting started with WDAC by Kim featuring Adam."
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2002-02-09
categories:
  - MEM
  - WDAC
  - WDAC
tags:
  - SCCM
  - ConfigMgr
  - Intune
  - MEM
  - WDAC
---


We've done quite a bit of Application Control work in the past months/years. One item that is always challenging in these types of projects is end-user communication. This post details one option for improving that communication.

https://twitter.com/TheWMIGuy/status/1554134893113778177
# The Backstory #
So Adam Juelich [Twitter]("https://twitter.com/acjuelich") had a conversation with Mike Danoski [Twitter]("https://twitter.com/MikeDanoski") about AppLocker in Intune. Somewhere along that conversation Adam mentioned WDAC, difficult and "Happy to be proven wrong, though" in one sentence, so here we are :-) 
 Adam has done a number of AppLocker implementations that were easier to get going than a WDAC implementation is considered to be. So, in this blogpost, we're going to try and mimick his AppLocker approach as good as we can with Windows Defender Application Control. 

 <blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">A lot of times I would implement AppLocker to primarily, initially, only allow certain Apps to be installed in the User Profile (user-based Applications).  That solves a lot of issues in a lot of environments.  That seems difficult to do in WDAC.  Happy to be proven wrong, though </p>&mdash; Adam Juelich (@@acjuelich) <a href="https://twitter.com/acjuelich/status/1554094399537160194">Aug 1, 2022</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

# Getting Started

## The Basics - Mimicking the Default Applocker rules
### Default Applocker Rules

### A Default Windows Defender Application Control Policy

## Adding some additional Path Rules

## Adding some publisher rules

## The differences between this "Classic" Applocker implementation and WDAC
 # The Why/ the tradeoffs #
 ## Why? ##
 Let's get to the Why question (Yes, Wally, I know, I hate why questions to, but people keep asking 'em)>
 Q: Why would you want to switch from AppLocker to Windows Defender Application control?
 A: There's a couple of reasions:
 
 1. Applocker is no longer enhanced upon. It's still present as a feature, but it won't evolve beyond what the feature offers now.
 2. Applocker is not considered a security feature by Microsoft's Security response Center.
    
    a. As a result known bypasses are not necessarily getting serviced (And malware authors do use those bypasses)
    b. Additionally, there's no bounty program for AppLocker, so there's less reason for researchers to investigate AppLocker
3. Wdac comes with the Managed Installer functionality explained in previous posts and all the benefits that brings.
4. WDAC comes with a recommended block rules list, maintained by Microsoft to keep bypasses at bay without requiring you to stay on top of every single known bypass.

## Tradeoffs ##
1. WDAC does not allow you to create different rules for different users. Either you trust the code to run or you don't trust the code to run. Who's running the code shouldn't influence that decision. From my point of view making different rules (Which AppLocker does by default for accounts with Admin privileges) is mostly done to keep your loudest/ most vocal and often most influential users at bay. They're however the most targetted users by adversaries as they typically have the most permissions to cause serious harm.

2. WDAC enforces DLL app control natively.





# Getting Started - Simulating Default Applocker rules #


# Getting Started - Add some publisher rules #



# The Solution #


![Actions](/images/WDACEventTask05.png)

![Actions](/images/WDACEventTask06.png)






















