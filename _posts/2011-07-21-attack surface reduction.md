---
title: "Attack surface reduction"
header:
  overlay_image: attacksurfacereduction.jpg
  teaser: attacksurfacereduction.jpg
author: Kim Oppalfens
date: 2020-11-17
categories:
  - Security
  - AttackSurfaceReduction
tags:
  - Security
  - AttackSurfaceReduction
---

The attack surface reduction principle should be top of mind for IT admins.

## Intro ##

What is meant by attack surface reduction - generally?

The name itself is pretty self-explanatory, the idea behind attack surface reduction is to eliminate vectors of attack by reducing the elements that can be attacked. Plenty of what is already done in the Infosec industry are often forms of attack surface reduction. The general idea, what isn't there, can't be compromised is a pretty strong defense.

My CISSP manual compares attack surface reduction to body armor. The more body armor you have, the less attack surface left. It's a workable analogy, they could've taken it further by giving you massive body armor making you ultra-secure, yet make it impossible for you to move around.

Unfortunately, IT systems are often still very permissive of what is allowed, leading to a vast attack surface. The reasons for that vary, from convenience, over lack of time considering what the attack surfaces are, and various others.
 
The rewards towards your security posture however are significant when you can reduce elements that can be attacked.

## Microsoft ##

What is meant by attack surface reduction - in Microsoft speak?

Microsoft is busy re-branding a ton of Windows 10 (Enterprise) features as Attack Surface Reduction. Not to be confused with Attack Surface Reduction rules/controls which are a sub-component of the Windows Defender Exploit Guard feature set. Part 1of the nice series on [demystifying Attack Surface reduction rules](https://techcommunity.microsoft.com/t5/microsoft-defender-atp/demystifying-attack-surface-reduction-rules-part-1/ba-p/1306420) refers to Attack surface reduction as an umbrella term for all the built-in and cloud-based security features in Windows 10.

Microsoft had bundled the following items under the Attack Surface reduction umbrella:
- Attack surface reduction rules: A set of rules that eliminates very popular elements of attack. Raning from privilege escalation over fishing through lateral movement.
- Application guard: Prevent phising attacks through webbrowsers and Microsoft Office.
- Windows Defender Application Control: A serious shift in security where applications are no longer allowed to run by default. But only apps on an approve list are tolerated.
- System guard: A system to protect the integrity of the boot process.
- Device Control: Control the USB devices allowed on a system.
- Windows Defender Exploit guard
- Network protection: Expand Microsoft Defender Smartscreen to all HTTP(s) traffic as opposed to just protecting browsers.
- Controlled Folder access: A mechanism to prevent folders from cryptolockers
- Web content filtering (ATP): A proxy without the proxy. Protect devices from accesssing undesirable webcontent.
- Windows Defender Antivirus with cloud delivered protection
- Windows Defender Firewall with advanced security

As you can see, the list is quite impressive.

## Personal opinion ##

 There's a ton of things we're already doing that can be considered attack surface reduction.

- Software updates: The item that probably makes every Top 3 security best practice, patch, patch, patch. Patches however typically eliminate the software from doing things they probably shouldn't have been able to do from the inception of the software.
- Principle of least privilege: The famous principle where you only grant the necessary permissions is often about making sure that a system can't be attacked by compromising an account with too much permissions.
- Removing local admin permissions: Several reports yearly point out that security vulnerabilities are less impactful if users are no local admins.

There are quite a number of things that can be done additionally to increase security as can be seen from that list above. 

So, What did I mean when I said attack surface reduction should be top of mind, and double so for folks that manage an organisation's systems management platform. I guess this part will be split up into 2 different sections in this series.

1. The items related to reducing the attack surface of your systems management platform. Given our experience over the years, items related to the Microsoft systems management platform Microsoft Endpoint Manager
2. Items where the systems management platform can assist in implementing Windows 10 security features that reduce attack surface.

In both these areas we are seeing evolutions and lack of movement that are either unfortunate or even counterproductive. The next couple of posts will talk to points we think can be improved in both these sections. 






