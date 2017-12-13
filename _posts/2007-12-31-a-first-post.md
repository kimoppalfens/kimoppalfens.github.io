---
title: "Collection based on pending reboot"
header:
  overlay_image: template1280x960.jpg
  teaser: Reboot_400x400.jpg
author: Tom Degreef
date: 2017-12-13
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

So Ryan asked on twitter whether you coud build a collection out of the Shiny new Pending Reboot column added in SCCM 1710.

# Twitter question #
<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">Can a collection be populated based on the pending reboot status?</p>&mdash; Ryan Engstrom (@ryandengstrom) <a href="https://twitter.com/ryandengstrom/status/940743161051336705?ref_src=twsrc%5Etfw">December 13, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

When @jarwidmark forwarded me the tweet I figured it would make an easy enough blogpost to answer, so here it goes:

Yes.

select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System  join sms_combineddeviceresources comb on comb.resourceid = sms_r_system.resourceid  where comb.clientstate <> 0

Is a collection query that picks up any machine with a pending restart.
0 means no reboot required, any non-o value means the machine has reported to SCCM that a pending reboot is waiting.

Other possible values are:
- 1 = Configuration Manager initiated reboot (Pkg/App install that triggers reboot etc...)
- 2 = Pending file rename (The classical reason for pending reboots, file in use while needing an update)
- 4 Windows Update (Reboot needed after install of software updates, wua agent initiated)
- 8 Windows Feature (You've added a Windows feature that needed a reboot).

That being said, the above values are bit flags, so a mixture of these can occur as well.
Figuring out what the values 6, 9 and 3 mean are left as an educational excercise. :)


