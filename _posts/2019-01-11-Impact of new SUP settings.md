---
title: "Software Update Point - Supersedence Rules"
header:
author: Tom Degreef
date: 2000-01-11
categories:
  - SUP

tags:
  - SUP
  - WSUS
  - Cleanup
  - Expired
---

Changed behavior of the "WSUS cleanup wizard"

# Supersedence Rules - new behavior since SCCM CB 1806 #

The behavior of supersedence for software updates has changed with the release of Config Manager 1806. This is equally true for the WSUS cleanup wizard if you enabled that option on your software update point.  
These changes have caught me off-guard and had impact in one of my environments. I figured I might not be the only one... So that's why we have this blogpost :)  

The [Microsoft documentation](https://docs.microsoft.com/en-us/sccm/sum/plan-design/plan-for-software-updates#BKMK_SupersedenceRules) notes the following for the supersedence rules :

Before Configuration Manager version 1806, when Configuration Manager sets a superseded software update to Expired, it doesn't set the update to Declined in WSUS. Clients continue to scan for an expired update until the update is declined manually or via a custom script. After Configuration Manager version 1806, Configuration Manager will also decline the superseded updates in WSUS.

For the cleanup task, the [following changes](https://docs.microsoft.com/en-us/sccm/sum/deploy-use/software-updates-maintenance#wsus-cleanup-behavior-starting-in-version-1806) have taken place :

Starting version 1806, the WSUS cleanup option occurs after every sync and does the following cleanup items:
* The Expired updates option for WSUS servers on CAS and primary sites. 
* Configuration Manager builds a list of superseded updates from its database. The list is based on the supersedence behavior in the Software Update Point component properties. 
..* The update configuration items meeting the supersedence behavior criteria are expired in the Configuration Manager console.
..* The updates are declined in WSUS for CAS and primary sites but not for secondary sites.
* A cleanup for software update configuration items in the Configuration Manager database occurs every seven days and removes unneeded updates from the console. 
..* This cleanup won't remove expired updates from the Configuration Manager console if they're currently deployed.

## New option in SCCM CB 1810 ##

While checking all the software update configuration options due to the issue I encountered with the other changes, I also noticed a new option being available in the Supersedence Rules section of the Software Update Point Properties

![alt]({{ site.url }}{{ site.baseurl }}/images/SUP1.jpg)

As you can see, we can now individually control supersedence behavior for feature and non-feature updates. I assume most of you still use task sequences for feature upgrades, so the impact of this change is currently still very limited I think.

# Possible impact of changes #

When we configure a software update process, we usually create different rings/waves. An example of such a process can be like this :

Collection Name | Software available time | Installation Deadline
---|---|---
Technical Pilot | ADR Run time (ADR) | ADR + 1 day
Business Pilot | ADR + 7 days | ADR + 10 days
Production Wave 1 | ADR + 14 days | ADR + 28 days
Production Wave 2 | ADR + 16 days | ADR + 30 days

Since I live in Europe, we use the "offset" feature to evaluate the ADR 1 day after patch tuesday (in the evening).  

The result here is that the people in Production wave 2 will have their software updates enforced 31 days after patch tuesday at 9PM,so basically the installation starts on the 32nd day after patch tuesday.  
At least in most environments I encounter, most end users will still wait for the deadline to have their software updates installed, although they had 2 weeks notice...

Until the changes in 1806, this was working fine, but after upgrading I noticed something strange.  
I was following up on the install base of the latest windows 10 cumulative update and the installation numbers weren't that high yet, but the deadline hadn't passed yet. That would be in the evening when most people were home, so basically, the next morning for them.

Upon checking the details the next day, I noticed something very very strange. Suddenly my cumulative update wasn't in my Software Update Group (SUG) anymore ! Wait? What ?!  
So I started to dig into the details, status messages just to see what happened. Did I make a mistake and removed it from the SUG? Did somebody else make any changes?  
I tracked the removal down to the point in time when the WSUS sync ran... That couldn't be a coincidence :)

So, what happened then? Well, it's a combination of things