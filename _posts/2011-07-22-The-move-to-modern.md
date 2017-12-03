---
title: "Template Post"
header:
  overlay_image: template1280x960.jpg
  teaser: template512x384.jpg
author: Kim Oppalfens
date: 2008-12-31
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

Header Line
My take on moving to modern and the Co-Management bridge
# Intro #
Unless you've been living under a rock since the release of Windows 10 you should've heard the hype around moving to Modern in the Microsoft Stack. In this post, which might turn into a post series, as this is a topic I can talk/write about quite a bit, I'll share my personal view on this.

## Why? ##
First of all let's try to get a feel as to why, as a customer you'd want to do this. Make no mistake, the majority of businesses (98.43% (* 78.93% of Statistics are invinted on the fly.)) have the desire to be as profitable as possible as their single most important goal. 
Anything not directly related to the bottom-line is a secondary goal at best that somehow must lead to the first goal. 
If you're still following along after that statement that means that evaluating a move to modern management has to either:

1) Increase your revenue or margin
2) Has to drive down cost (Or avoid a sudden increase in cost.)
3) Has to fullfil a seconary goal that leads into items 1, 2 or a mixture of both

I can't immediately see how moving to modern is going to directly result into 1, and moving to modern in and of itself isn't going to drive down costs that I can see as an Intune license is generally more expensive than an on-prem managed license. The only way I can see 2 happening is by letting systems managers go, which conflicts with the talks that I strongly believe in that moving to the cloud won't cost people their jobs, it'll just mean their jobs become different.(There's a small financial benefit in not needing infrastructure, but the license cost of a systems management solution, combined with the man-power to operate it has always far exceeded that.)

Which leads my to my first conclusion that moving to Modern has to fullfil a secondary goal, that eventually leads into 1) and/or 2)

What that secondary goal is isn't all that difficult to see. It's the promise of working anywhere on any device that is aimed at making people more productive, which should lead to 1). And could help with driving down Office space cost.

This leads me to a second conclusion, a move to Modern's success, as a business is measured in it's ability to deliver on the promise to work from anywhere and as side-objective on any device. I refer to the any device as a secondary KPI as again, in my eyes, there are classes of devices:
- Laptops
- Desktops
- Thin clients
- Tablets
- Smart Phones
- HoloLens
- ...

Certain workloads are just better suited for a certain type of device class, and trying to make, or letting people do a workload on the wrong device doesn't make them more productive. (It might make them more happy, which you could stretch into reaching objective 1) or 2) but that's always hard to quantify. The part I do strongly agree with is that no matter what class of device you work on, the underlying OS on that device should be immaterial. Again, if you're accustomed to working on MacOS, it's going to be hard to make you more productive on a Linux or Windows device and vice versa.

So in summary, if you're move to Modern has failed on delivering on the worf-from-anywhere-promise, I consider it to have failed. Additionally, if you as a company don't belief in a work from anywhere world, than moving to Modern makes no sense for you. (And I consider you to have failed as a company as well.)


## How? ##
The move to Modern, as can be seen on the freshly releaded info-graphic about Co-Management, comes with moving workloads to new tools. Now, there's 2 ways to go about this.

1) Starting fresh, the greenfield approach, building from the ground-up or whatever you want to call it.
2) Migrate the current workloads.

### Starting fresh ###
Starting fresh doesn't take any of the current practices into account and completely re-defines what is needed on a device of the future. This is definitely the quickest way to move to Modern, but introduces a number of risks. 
1) Without evaluating current practices you might miss important items that are well, important.
2) In this type of setup, the needs are all too often based upon what is feasible by the technology, as opposed to what is necessary from a company perspective. (This has for instance lead to the acceptance of users being local administrators again in Modern for quite a while. (Believe me, that is still a terrible idea, and taking shortcuts on security like this will one day conflict with item 2) in the introduction, guaranteed.)

If you plan on using the "Starting fresh" method, be aware of these risks, and don't allow a "pragmatic" approach to undermine good security practices.
Even more important, chose your target audience and devices wisely. Trying to do a "Starting fresh" method company-wide from the get-go has a slim chance of being a success for most businesses. 

Additionally if you think of the move to Modern as a switch in technology, you're most likely setting yourself up for failure.

### Migration of workloads ###
Evaluating what is in place now, consolidate, and move the workload is a different way of handling the move of workloads.

This requires a far more advanced and methodological approach, and as with starting fresh it requires a good selection of good potentical candidates to Pilot the move to Modern. This approach requires decent Information on what is still needed, followed by an evaluation of what can be delivered in the Modern approach. I spelled Information with a capital I, because that's what's needed to be successful in this approach, Information, not data.

As an example of what I mean by that I give you the following. Let's say you have a couple of applications where you currently use Group Policies to manage it's settings. Knowing who uses these applications on what device is an important data-point to have to decide how to handle this migration. That data can be gathered, but isn't result-driven enough as an approach. Typically people would then use that data to come up with solutions for the app's used on most devices. Transforming that data into Information would mean you analyse the data and look at the application that unblocks most users / machines to move to Modern because it's the single, or part of a small number of applications managed in this way that still blocks them. Using the latter approach might result in a very different priority list than using the former.