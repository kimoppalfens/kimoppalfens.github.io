---
title: "Reducing attack surface with Application Control and managed installer(s)"
header:
  overlay_image: template1280x960.jpg
  teaser: template512x384.jpg
author: Kim Oppalfens
date: 2008-12-31
categories:
  - AttackSurfaceReduction
tags:
  - SCCM
  - ConfigMgr
  - Intune
  - Security
  - AttacksurfaceReduction
---

This post will explain the basics of how a Windows Defender Application Control managed installer works. In subsequent posts we'll start defining managed installers for Intune & MEMCM.

# Intro #

This is another post in the attack surface reduction series. In this installment let's start discussing application control. Application control peaked my interest a couple of years ago and that interest sky rocketted when I heard about the managed installer functionality introduced in Windows 10 1709.

Apart from the public [docs at microsoft](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/windows-defender-application-control) the goto resource on Wdac is [Matt Graeber aka @mattifestation](https://twitter.com/mattifestation?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor). I've learned a ton from his many posts on the topic.

Between Matt's posts and the docs there's quite a lot out there on Wdac itself. One thing severily lacking, other than frustrations uthered in several forums, is how you manage and maintain this in an environment that's under the control of a systems management solution. That's where managed installers come in.  I'll try to fill in some of the detail around that.

## The basics ##
Managed installers are supposed to make all software installed through your Systems Management platform trusted by your Application Control policy. Given my background in this area (I have been and still am an MVP for 15 years in this field of expertise) the platforms in scope are Configuration Manager and Intune. So everything deployed by either of those solutions would pass your Application Control policy and be allowed.
Again, working in this field the end goal has always been to install every single thing needed through that centralized platform as this eliminates the need for admin accounts to install things.

So how does this work? Whenever something is downloaded and/or installed by a managed installer, in other words whenever something is written to disk by a Managed Installer a "tag" is left on that file detailing the originating process that wrote the file to disk. The "tag" uses an NTFS feature called extended attributes to store that data. The somewhat peculiar thing about Managed Installers is that they have been implemented through a mix of Microsoft Applocker settings & Windows Defender Application Control. You define the rules for Managed Installers in Microsoft Applocker AND you specify a ruleoption in Windows Defender Application Control that you want the Managed Installers defined to pass your WDAC checks.

That's roughly all there is to it. The tagging of the files works just fine while your WDAC policy is still in audit mode. In other words, you can do your future self, or anyone following you in the sysadmin a huge favour by just enabling it now.

If you think through what I wrote above for a minute the conclusion is that Managed Installers do nothing for software and files written to disk prior to defining your Managed Installers. The files written to disk weren't tagged at the time they were written to disk and there's no way to tag them after the fact. I assume it is becoming more clear that enabling this now might be hugely beneficial down the road.





