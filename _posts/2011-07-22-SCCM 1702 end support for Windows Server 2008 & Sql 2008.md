---
title: "What does SCCM End of Support For SQL & Windows 2008 Versions really mean?"
header:
  overlay_image: clock1280x960.jpg
  teaser: clock512x384.jpg
author: Kim Oppalfens
date: 2017-03-30
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

As Announced, SCCM Current branch 1702 no longer supports the 2008 editions of SQL or Windows. What does this mean if you're SCCM infrastructure is still running on-top of eather of those?

# Intro #
It shouldn't come as a surprise to people that follow the developments of SCCM from close-by. The deprecation of support for the 2008 versions was mentioned first on July 10th 2015. And has now been reconfirmed with the actual release of the first current branch in 2017. [https://blogs.technet.microsoft.com/configurationmgr/2017/03/10/configmgr-current-branch-support-removal-reminder/](https://blogs.technet.microsoft.com/configurationmgr/2017/03/10/configmgr-current-branch-support-removal-reminder/)

Again, those that follow this product's developments should be aware that an in-place upgrade from 2012 R2 to SCCM Current branch is the preferred method of moving to an updated infrastructure. This approach comes highly recommended by the product team. And they've invested quite a bit in this, by doing a ton of tests to follow-up the SCCM upgrade with an OS upgrade and a SQL upgrade.

Things you should be aware off:
- The product team has invested a ton of time in making upgrades as easy as possible.
- Upgrading from a 2012 edition only works towards what is called a baseline build.
- ConfigMgr current branch 1606 is the latest baseline build that still supports the 2008 editions.
- ConfigMgr current branch is the CM version that supports an in-place upgrade of the Operating System.
- A ConfigMgr build is supported for 1 year after release.

If this is news to you, and/or your infrastructure is still relying on one of these components, the next portion of this post highlights your options. 

## SCCM 2012 running on Windows 2008 ##
If you're still running on Windows 2008:
Why on earth haven't you moved yet? Seriously, a ton of customers have upgraded before you. The 2012 build doesn't support deploying the Windows 10 Anniversary update, creators update or any future Windows 10 builds for that matter.

Given the details above, and past experiences from tons of customers worldwide, even a couple of pretty big ones, you should strongly consider upgrading quickly. Looking through the details above the clock is ticking. By June/July of 2017 you'll have to have upgraded to the Current Branch edition if you still wish to use the in-place upgrade scenario. Failing to do so would mean that you're only option is doing a side-by-side migration which will add a considerable amount of time to your transition.  

## SCCM 2012 running on SQL 2008 ## 
SCCM 2012 has always supported moving the database to our more recent supported SQL server version. If you're underlying OS is Server 2012 or later, the 1610 build still supports the SQL 2008 edition. So you have a bit more time here to get this upgrade done. October/November 2017 is the timeframe at which point this should have been done.

## SCCM Current branch version running on Windows 2008 ##
If you've already performed the upgrade to Current branch, but haven't performed the OS upgrade yet, or made the unfortunate decision to install Current branch on top of Server 2008, than you have till October/November 2017 to make the OS Upgrade happen.


## SCCM Current branch version running on SQL 2008 ##
See SCCM 2012 running on SQL 2008, the story here is identical.
