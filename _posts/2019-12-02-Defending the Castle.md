---
title: "Defending the (SCCM) Castle"
header:
author: Tom Degreef
date: 2010-12-02 
categories:
  - SCCM

tags:
  - SCCM
  - Configuration Manager
  - Security
---

With great power comes great responsibility !

# Intro #

To start of with the words of a wise man :

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending01.jpg)

Who am I to question spiderman? 
The fact is that our beloved Configmgr, in most environments, manages all devices. Both servers and workstations. 
Every action taken by Configmgr is executed with the highest privileges.

If I somehow manage to compromise your Configmgr environment and make it execute some malicious code, would you notice ?
Extra (network) traffic it would generate will probably not stand out as it would look like regular Configmgr traffic.

Even [Windows Defender Application Control](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/windows-defender-application-control) will probably not even protect you at this point because chances are that you listed Configmgr as a "managed installed", as such trusting everything it executes.
**note:** The above statement should not be used as an excuse not to implement application whitelisting! We still strongly believe that application whitelisting one of the [best defences against malware](http://www.oscc.be/osccservices/Windows-Defender-Application-Control/).

The point really is that you should not take security on your Configmgr environment lightly, because it might be very difficult to detect abuse.

We have already spent 2 ([1](https://sched.co/N6f0),[2](https://sched.co/N6f3)) full sessions at MMSMOA 2019 on this topic [1](https://sched.co/N6f0) [2](https://sched.co/N6f3) and a "demo-only" session at MMS Jazz Edition. In Belgium we presented this at Lowlands Unite 2019.

This blog post is meant as a summary for what we have shown before.

## SQL Security ##

## Status Filter Rules ##

## Attacking Client Push ##

## Decyphering the Network Access Account ##

## Hiding applications in your Admin UI ##

# Conclusion #

