---
title: "WDAC for the win #73 - The one with Nvidia"
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2022-03-09
categories:
  - WDAC
tags:
  - WDAC
  - ConfigMgr
  - Intune
  - MEM
---

This post details howto implement a Wdac policy to block the stolen Nvidia certs.

# Intro #

Early in March a Twitter conversation happened between a number of people regarding the Nvidia security incident that involved the leakage of some of Nvidia's expired codesigning certificates. This evolved into a recommendation by David Weston to use #WDAC as a first line of defense. You can find the twitter conversation below.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">WDAC policies work on both 10-11 with no hardware requirements down to the home SKU despite some FUD misinformation i have seen so it should be your first choice.  Create a policy with the Wizard and then add a deny rule or allow specific versions of Nvidia if you needpic.twitter.com/gcNLJoJ0Gn</a></p>&mdash; David Weston (DWIZZZLE) (@dwizzzleMSFT) <a href="https://twitter.com/dwizzzleMSFT/status/1499527802382471188">March 4, 2022</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## The basics ##

Creating a deny rule is something that can definitely be done in Application Control. To do that though, as for most signature based rules you need something that's called a TBS Hash. (If I am not mistaken, TBS stands for To Be Signed and it's a hash based representation of the codesigning cert.) 

The annoying bit is that you need either a file signed with the certificate or the certificate itself to be able to create that rule. That's why it took me a while to build the rule. I first had to get access to the certificate somehow.

## Building the Policy ##
I received a copy of one of the certs today, so decided to build the policy and share it out. I created the policy using the following PowerShell command.
```Posh
Add-SignerRule -FilePath .\DefaultWindows_Audit.xml -Deny -CertificatePath .\nvidia.cer
```

## Item 3 ##








