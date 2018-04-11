---
title: "Moving to Modern management - Kim's personal opinion"
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

The Co-Management bridge and moving to Modern.
# Intro #
Unless you've been living under a rock since the release of Windows 10 you should've heard the hype around moving to Modern in the Microsoft Stack. In this post, which might turn into a post series, as this is a topic I can talk/write about quite a bit, I'll share my personal view on this.

## What? ##
If you only remember a single thing from this post, let it be the mantra "Define it, define it, define it!"

The term modern is used all-over in our field of work the past months and years, and I have yet to see a comprehensive agreed definition that everyone adheres to. Quite often it is defined as changing tools and using Windows 10's configuration service provider to do your management. Which often comes down to changing your tools and managing your Windows 10 devices as if they were phones or laptops. I can't say I am happy with that explanation as changing tools just for the sake of changing tools is seldomly a sensible idea. 

### My definition (And some comments) ###
So I'll tell you what I personally think Modern should be about:
- allowing people to work from anywhere
- allowing people to easily switch from one device to another
- allowing people to be as productive as possible (in a secure manner)
Those are the 3 corner stones in my definition, in order of importance, with item 1 far outweighing anything else. 

This definition avoids talking about tools to do the job, and importantly, avoids a cyclic description by re-using the word modern. I'll go into detail about these items later in this post, and what I consider them to mean. Oh, and the blog does have a discussion section, so feel free to discus, agree, or give me your definition.

### Other descriptions (And my comments) ###
[Manage Windows 10 in your organization - transitioning to modern management](https://docs.microsoft.com/en-us/windows/client-management/manage-windows-10-in-your-organization-modern-management)
> Use of personal devices for work, as well as employees working outside the office, may be changing how your organization manages devices.

This description explains modern device managment as being based on the user of personal devices, and employees working outside the office. As you can see above, I fully agree with the second one. As to the use of personal devices for work, yes, I see that happen. However, I largely see it happening for phones, a bit for tablets, and a neglectable amount for laptops. And if I do see it happen for laptops, that laptop typically isn't running Windows 10.

[Modern management for Windows 10](https://blogs.technet.microsoft.com/windowsitpro/2016/12/15/modern-management-for-windows-10/)

> Earlier this year, we shared our vision for “modern management” for Windows 10, in which organizations can manage Windows 10 devices of any form factor – PC, tablet, phone, HoloLens, Surface Hub etc. – in the same manner as the rest of the mobile devices in their environment.
- Using mobile device management (MDM), organizations can more easily manage Windows devices, regardless of whether they are on the corporate network or not. Managing through MDM also helps address today’s more common issues. There are more mobile workers—many of whom don’t work out of a corporate office.
- Users want the flexibility to use their own personal devices.
- IT pros are challenged with the task of managing devices in a mobile-focused, mixed-use device environment while maintaining productivity for users, protecting corporate assets, and keeping costs low.

1. This description specifies a vision to manage all Windows 10 devices just like other mobile devices in the envirionment. My objections to that is that a typical Windows 10 machine isn't like other mobile devices. A Windows 10 device can run code from anywhere, and is in essence jailbroken by design. As a result, especially in this hostile online era, they have different requirements. Managing them with a lite touch as you manage a phone or a tablet can work, when we are talking about devices that are protected from running malicious code, like Windows 10 S, or a device that has an application whitelisting solution implemented like Device guard, but that is a minority of devices.
2. There is the mobile work from anywhere part again, which I fully agree with.
3. Nope, as stated above, there is no huge push from users to use their own personal Windows 10 devices. Not at the customers I meet, and not by the people I talk to. That push exists largely for phones, and people want to easily transition work flows from phone to laptop. And some people want a way easier experience when their laptop needs replacing for whatever reason (Lifecycly, broken laptop, forgotten laptop...)
4. Yes, that is indeed the challenge for IT pros summed up quite nicely.

## Why? ##

First of all let's try to get a feel as to why, as a customer you'd want to do this. Make no mistake, the majority of businesses (98.43% (* 78.93% of Statistics are invinted on the fly.)) have the desire to be as profitable as possible as their single most important goal. 
Anything not directly related to the bottom-line is a secondary goal at best that somehow must lead to the first goal. 
If you're still following along after that statement that means that evaluating a move to modern management has to either:

1) Increase your revenue or margin
2) Has to drive down cost (Or avoid a sudden increase in cost.)
3) Has to fullfil a seconary goal that leads into items 1, 2 or a mixture of both

I can't immediately see how moving to modern is going to directly result into 1, and moving to modern in and of itself isn't going to drive down costs that I can see as an Intune license is generally more expensive than an on-prem managed license for organizations that don't already own Intune licenses for their phone and tablet management needs. The only way I can see this happening is by letting systems managers go, which conflicts with the talks that I strongly believe in that moving to the cloud won't cost people their jobs, it'll just mean their jobs become different.(There's a small financial benefit in not needing infrastructure, but the license cost of a systems management solution, combined with the man-power to operate it has always far exceeded that.)

Which leads me to my first conclusion that moving to Modern has to fullfil a secondary goal, that eventually leads into 1) and/or 2)

What that secondary goal is isn't all that difficult to see. It's the promise of working anywhere on any device that is aimed at making people more productive, which should lead to 1), and could help with driving down Office space cost.

This leads me to a second conclusion, a move to Modern's success, as a business is measured in it's ability to deliver on the promise to work from anywhere and as side-objective to easily transition workflow between devices (Which is vastly different from the work on any device statement). 

There are classes of devices:
- Laptops
- Desktops
- Thin clients
- Tablets
- Smart Phones
- HoloLens
- ...
Certain workloads are just better suited for a certain type of device class, and trying to make, or letting people do a workload on the wrong device doesn't make them more productive. (It might make them more happy, which you could stretch into reaching objective 1) or 2) but that's always hard to quantify. The part I do strongly agree with is that no matter what class of device you work on, the underlying OS on that device should be immaterial. Again, if you're accustomed to working on MacOS, it's going to be hard to make you more productive on a Linux or Windows device and vice versa.

So in summary, if you're move to Modern has failed on delivering on the work-from-anywhere-promise, I consider it to have failed. Additionally, if you as a company don't belief in a work from anywhere world, than moving to Modern makes no sense for you. (And I consider you to have failed as a company as well.)


## How? ##
The move to Modern, as can be seen on the freshly releaded info-graphic about Co-Management, comes with moving workloads to new tools, but it starts by implementing Co-management itself. Which for Configuration Manager clients includes deploying the Configuration Manager cloud management gateway. That part gets us a long way towards being managed from anywhere.

Implementing co-management on top of that gives you seem immediate benefits, even without transfering workloads.

My esteemed MVP colleague Panu Sauko had a Co-Management session at Techdays Finland. [Co_Management session at Techdays Finland](https://onedrive.live.com/?authkey=%21AMBvF68YeroDTBE&cid=6F87805E6B839EB2&id=6F87805E6B839EB2%21117&parId=6F87805E6B839EB2%21104&o=OneUp)

In there he makes a couple of good points. 
1. Do you want to transition to an end-goal of having only the Intune client? Or do want to stick to the "Bridge" offering the flexibility of the full Configuration Manager client from anywhere by using the Cloud Management gateway.
2. A list of benefits you immediately get when implementing co-management on top of the Cloud Management gateway without transfering any workloads
- Devices synced to the Azure Portal
- Execute remote actions (Factory reset, Selective wipe, Delete devices, Restart device, Fresh start)

And another colleague, Tom Degreef, did a series on how to implement Co-Management right here:
[http://www.oscc.be/tags/#co-management](http://www.oscc.be/tags/#co-management)

## Summary ##
The Bridge, Co-Management, the Configuration Manager Cloud management gateway, they're all pieces of a puzzle to get you to a device that can be managed from anywhere. That being said, a device being manageable from anywhere doesn't necessarily mean you can actually perform work on it from anywhere.

That requires a set of different puzzle pieces:
- Access to productivity tools that exist on the laptop, tablet and phone (Eg: Office)
- Access to files on the laptop, tablet and phone (Eg: Cloud storage like Onedrive for business)
- Ability to authenticate from anywhere (EG. A Cloud identity like Azure Active Directory)
- Ability to access onpremise resource

To sum it all up, and making it product specific this time around, my prefered Modern managed device has:
- a Configuration Manager client that is configured to talk to a cloud management gateway
- Is Hybrid Azure Ad joined
- Has the Windows 10 security stack implemented
- Has onprem applications made available to it using an Azure AD proxy
- Is enrolled in Conditional Access 
- Is enrolled in co-management

And a device like this, preferably, is managed by folks that implement a great deal of self-servicing using the tools at their disposal. 

Those same Configuration Managers, additionaly, avoid medling with settings that restrict users and that don't have a solid compliance reason. As an IT Pro we're typically not good at deciding what is the most convenient way to configure settings for an end user, and there's no reason for us to get involved.

If you want a properly managed Modern Windows 10 device, that can be managed from anywhere. Took a real hard look at the functionality provided by a Configuration Manager client with a Cloud Managemen gateway and compare that to whatever else you had in mind. Keeping in mind that it's not about the tool, but about the functionalities you need, and the functionalities delivered by your choosen solution.

**PS**: Intune licenses offer you the right to install Configuration Manager. If you're managing Windows 10 devices using Intune only now, you might want to take a look at Co-Management and extend your setup with a second, very flexible, systems management agent.

Discus!!!





