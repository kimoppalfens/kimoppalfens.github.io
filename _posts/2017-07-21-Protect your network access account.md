---
title: "Protect Network Access Account"
header:
  overlay_image: NetworkAccessAccount1280x960.png
  teaser: NetworkAccessAccount512x384.png
author: Kim Oppalfens
date: 2008-12-31
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

An over-priviliged network access account is a security risk specific to SCCM environments. This post is about protecting that account.

# Intro #
The network access account is an account that is used in most SCCM environments. It is used to grant access to distribution point content by machines that can't use their domain computer account to authenticate agains the distribution point.
The new SCCM Current branch docs site describes it [like this](https://docs.microsoft.com/en-us/sccm/core/plan-design/hierarchy/manage-accounts-to-access-content).

>Client computers use the Network Access Account when they cannot use their local computer account to access content on distribution points. For example, this applies to workgroup clients and computers from untrusted domains. This account might also be used during operating system deployment when the computer installing the operating system does not yet have a computer account on the domain.

One of my fellow MVP's, Roger Zander, has described it as [evil.](http://myitforum.com/myitforumwp/2015/05/11/network-access-accounts-are-evil/).



