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


Application Control will most likely still be one of the major ways to combat malware in 2023. We've seen and helped implement quite a bit of implementations, including a significate number of Applocker ones. Than Twitter happened back in August and I promised I'd write something about that.

# The Backstory #
So Adam Juelich [Twitter]("https://twitter.com/acjuelich") had a conversation with Mike Danoski [Twitter]("https://twitter.com/MikeDanoski") about AppLocker in Intune. Somewhere along that conversation Adam mentioned WDAC, difficult and "Happy to be proven wrong, though" in one sentence, so here we are :-) 

 Adam has done a number of AppLocker implementations that were easier to get going than a WDAC implementation is considered to be. So, in this blogpost, we're going to try and mimick his AppLocker approach as good as we can with Windows Defender Application Control. The first post will show you how to get a policy with similar roles. I'll follow it up with some behavioral differences and how to get as close to the Applocker behaviour as possible next week.

 <blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">A lot of times I would implement AppLocker to primarily, initially, only allow certain Apps to be installed in the User Profile (user-based Applications).  That solves a lot of issues in a lot of environments.  That seems difficult to do in WDAC.  Happy to be proven wrong, though </p>&mdash; Adam Juelich (@@acjuelich) <a href="https://twitter.com/acjuelich/status/1554094399537160194">Aug 1, 2022</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

# Getting Started

## The Basics - Mimicking the Default Applocker rules
### Default Applocker Rules
Applocker comes with the ability to define default rules. The rules in essence make all your well-behaved applications and Windows work and disable Applocker for Administrators. These rules exist for the 5 diffent Applocker Rule Collections.
- Exe (Executable Rules)
- Dll (DLL Rules)
- Msi (WIndows Installer Rules)
- Script (Script Rules)
- Appx (Packaged app Rules)

The Default rules for Appx are somewhat different in that all signed appx's are trusted, no matter where they run from or whether they run in Administrator context.
This immediately poses some differences that we'll get back to in the following post.
1. WDAC does not have a way to differentiate its policies based on different users.
2. WDAC does not have a concept of different rule collections in the same way that Applocker has.

### A Default Windows Defender Application Control Policy
WDAC does have a concept of a  Default policy too, although here they're called ExamplePolicies as opposed to default. One of the reasons I can understand why people call Applocker easier is that you kind off have to search for these Example policies. They are stored in your Windows folder in .xml format in the following path: c:\windows\schemas\CodeIntegrity\ExamplePolicies.
You should find 6 ExamplePolicies to start from:
* AllowAll.xml

Description: Somewhat self-explanatory with a twist. The AllowAll policy trusts every single file and so allows everything to run. So, what's the twist? Well, files whose code signing signature validation fails should still be stopped by this policy.
* AllowAll_EnableHVCI.xml 

Description: Identical to the AllowAll policy but equally enables HyperVisor CodeIntegrity
* AllowMicrosoft.xml

Description: As the name suggests, this should trust all code codesigned by Microsoft. Somewhat more challenging to verify as this policy lists a number of approved signers. How well this works depends on the completeness of said list.
* DefaultWindows_Audit.xml

Description: This is the policy we start from more often and from my point of view can be considered the Default policy, hence it's name. It is quite different from the Default Applocker rules though. Whereas the default Applocker rules make most well-behaved software trusted on top of Windows itself, this policy mainly makes the Windows OS trusted. There are some non-Windows things that in our experience become trusted too.
- Office 365
- Teams
- Microsoft Edge
- The Intune Management Extensions
- Windows Defender
- The Windows Defender for Endpoint (mostly)


* DefaultWindows_Enforced.xml

Descrption: Identical, but with the rules enforced instead of in Audit mode.
* DenyAllAudit.xml

Description: Blocks all execution, only sensible to do in audit mode, if at all.


### Adding some additional Path Rules

### Adding some publisher rules

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






















