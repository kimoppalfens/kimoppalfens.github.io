---
title: "(Un-)Expected Software Update Point behavior"
header:
author: Tom Degreef
date: 2007-09-28
categories:
  - SCCM
  - Configmgr
  - SUP
  - Windows 10
  - 1706

tags:
  - SCCM
  - Configmgr
  - SUP
  - Windows 10
  - 1706
---

(Un-)Expected Software Update Point behavior

# Intro #

While working recently on a project that involved an SCCM migration to a new 1706 environment, we noticed that the newly installed clients in this 1706 environment didn't download any software updates.

As a first troubleshooting-step we started looking in the server-side logs as all clients seemed affected, but they were all clean, not a single error to see. Same for client logs... all very clean ! We did however notice that no software update point was referenced in any of the logs ...

# Environment layout #

To dig deeper I need to give you an idea of their environment. It's a fairly simple setup with a single primary site and all components one the primary site server. (including the software update point)
In addition to that, they also have 4 distribution points to accomodate their remote offices.

Boundares are Ip-Range boundaries and a boundary group per distribution point and location. An additional boundary group was created for the internet-based clients and that included the DP on the primary site server itself (and thus the SUP)

# Issue #

Since Configmgr 1702, **New clients** use boundary groups to select software update points. This replaces the previous behavior where clients select a software update point randomly from a list of those that share the client’s forest.

In our case, since none of the on-prem clients had a SUP in their boundary group... no software update point could be located. Boundary/Boundary group configuration was identical to their old environment but those clients were updated from "older" versions and thus they didn't suffer from this behavior. In the new 1706 environment, it were all newly installed clients.

Now in my particular case, we didn't want to add the software update point to the boundary groups of the remote location so we specified the fallback behavior on the remote location boundary groups to only fallback for the software update point and not for the DP.

![alt]({{ site.url }}{{ site.baseurl }}/images/fallback_sup.PNG)

Once that was configured and clients updated their policies, software updates began to install again.

# Conclusion #

While reviewing the documentation for 1702 (and 1706) I noticed that this is indeed properly [documented by Microsoft](https://docs.microsoft.com/en-us/sccm/core/servers/deploy/configure/boundary-groups#software-update-points), so it didn't sink in while reading the release notes.
However, it occured to me that this might be the same for other Configmgr Administrators ;-)  Hence this blog-post.

Basically what I am saying is, make sure to review your boundary group configuration if you are on SCCM 1702 (or newer). Existing clients will keep working but any newly installed machine (or reinstalled client) will be affected by this behavior and it will be difficult to spot in your software update reports.