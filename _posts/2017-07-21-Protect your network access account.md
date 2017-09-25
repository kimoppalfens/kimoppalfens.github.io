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

### The security risk
One of my fellow MVP's, Roger Zander, has described it as [evil.](http://myitforum.com/myitforumwp/2015/05/11/network-access-accounts-are-evil/).
And he makes a good point, you really should treat the Network Access account as an account who's password is compromised. That means you have to configure it so that it can only do specifically what it is desgined for, authenticate against a distribution point.

The docs site, quite recently, strongly specifies, in color that the account should NOT have interactive login rights, AND should NOT be re-used as your account to join machines to a domain in your task sequence. My personal recommendation for it is to not re-use it for any other purposes. Again, the docs site specifies which user rights the account needs, and that is "Access this computer from the network" on distribution points.

### Protecting your Network Access Account - Explained
Based on that I have create a number of CI's and a baseline that allows you to easily make sure the Network Access account is limited to exactly these rights [here.](http://www.oscc.be/sccm/configmgr/Protecting-the-Network-Access-Account-using-Configuration-Items-Quest-Part1/)

Implementing that script however required you to create your own ci's copy the script in and mofify it with your own NAA accounts. I got feedback that this was too cumbersome for quite a number of admins. So in this post I'll share my extracted baseline alongside a PowerShell script that does the following:
1. Import the Configuration Baseline
2. Read the NAA Accounts definied in your environment
3. Adapt the CI scripts to configure your Network Access account protection

The CI's themselves configure all your clients with the following deny user rights:
- Deny Logon as a service
- Deny Logon as a batch job
- Deny logon on locally
- Deny logon through Remote Desktop Services
- Deny access this computer from the network (Should only be applied  to non-distribution points)

###Implementing  the solution







