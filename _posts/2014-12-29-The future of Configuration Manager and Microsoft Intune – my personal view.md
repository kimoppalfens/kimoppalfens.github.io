---
title: "The future of Configuration Manager and Microsoft Intune – my personal view"
author: Kim Oppalfens
date: 2014-12-29
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

This is a blog post that is long overdue from my part, as I've been willing to write this for a long time. I will however start off with a "disclaimer" on this one. These are my personal views, and although I've been rewarded the Enterprise Client Management MVP for 10 years, or a decade if you will, none of this is based on inside information I've received from the product group.  


## Current State of affairs

 

### Intune Standalone

  
Intune standalone tends to receive new feature first, Microsoft has a Cloud first and Mobile First strategy. Microsoft Intune, which as I'll attest in this post, is largely a mobile device management solution seems a natural fit for that. A couple of important things are happening for Intune, significant changes are going around protecting the data with EMS including Rights Management, and Multi factor authentication being added. Combined with an alignment of the device management enrollment experience accross the different mobile platform are important steps for the future of Microsoft Intune. I still feel steps need to be set to make managing the online identities easier, granted the newly released Azure Active directory sync services seems to be a good step in the right direction at that.  

### Hybrid Intune

  
Hybrid Intune, integrated into System Center Configuration Manager 2012 R2 usually receives features released for Intune Standalone some time after the standalone release. Some features follow quickly, some take a little more time.  

## Heavy Investments in Microsoft Intune explained

  

### Clarification Heavy investments in extending its mobile capabilities

  
To be clear, a lot of the ConfigMgr administrators aren't particularly happy with how much investment goes to the "cloud", and what seems to be a fraction going to the "on-premise" stuff. Quite a couple of them seem to conclude that Microsoft is aiming at replacing Configuration Manager. When you look at the investments though, most if not all of these investments are being done in features related to mobile device management. In fact, quite a few companies that were onboard with Windows Intune in the days it was still named that, and was originally geared as a systems management solution for SMB are disappointed for the the lack of progression made in that field in the last 3 years or so. Nothing new has been done for that market in a long long time, yet this might add to the confusion as the original Intune was seen as a replacement for System Center Essentials whose development was stopped not so long before Windows Intune surfaced. In my view, Microsoft completely repurposed the Windows Intune infrastructure / architecture for mobile device management. They didn't go as far as to eliminate the workstation management features already in it, but other than sustaining the code I've yet to see huge improvements in that particular field. If they did happen, I definitely have missed them. So, yes, Microsoft has a cloud focus, however, as being stated by Microsoft plenty of times it's Cloud Only, it's Cloud first. Which, is to my point, was recently changed to Cloud First, Mobile First.  

### Mobile device market booming like crazy

  
The level of investments in the mobile device management market are largely because the mobile device market itself is booming like crazy. Again, some people seem to draw the conclusion that this means the end of the regular windows market. Again, when you look at today's numbers, the regular Windows market is nothing shy of steady for the next 3 years according to the same predictions used by Brad Anderson in his post here:  There's no huge increase in sales number, yet there's no significant decrease or collapse neither. When looking at IDC's latest forecasts, which is the same source Microsoft used, the numbers are actually still increasing: . Less desktops, largely compensated by higher lapto sales, given that some of those tablets. With the success of the Surface pro 3 part of that tablet market is in the category of regular Windows as well. The mobile device market on the other hand has a projected potential of becoming anywhere between 3 to 5 times the size of the regular Windows market. On top of that number of devices game, there is still a challenge in convincing/finding on offer to convince quite a few businesses that mobile device management is needed. I guess I am kicking in an open door when I say that systems management of mobile devices requires a vastly different feature set than that of managing non-mobile devices.  

### Systems management in mobile device market undergoing rapid changes as well

  
The other reason Intune is drawing in so many investments / is costing so much money is that the systems management field is offering management of multiple platforms, which increases the effort that has to be put in. And every new Mobile Os comes with increased abilities for management that need to be supported. This combined with the rapid pace at which new versions of Mobile platforms are released mean you need a big team to keep up. In the past couple of years though, keeping up was far from enough for Microsoft. They were new in the market and had to play catch-up feature wise big time. With the investments done in the past 2 to 3 years, and the release of EMS Microsoft is finally closing the gap somewhat.  

## Requirements for Intune to take over the world

  
You'll notice a theme in this section on things I consider a requirement before Microsoft Intune can become the Systems Management tool to rule them all. I believe you need at least 4 features for a systems management solution that Microsoft Intune standalone at present doesn't offer, or where it needs extensive work to provide something competitive.  

  

1. OS Deployment
  

2. Software distribution
  

3. Server Management
  

4. License management
  
  
NOTE: These 4 features are needed in my perspective for what me and a colleague of mine have starting to name "Open devices". To explain systems management in today's world we've chosen to kitagorize devices as open versus closed, instead of using mobile devices, hybrids, laptops, desktops, etc….

Open devices to us are devices that have alternate means of installing software outside of a controlled store, whereas closed devices are devices that can only install software from the corresponding store.  

### OS Deployment

  
OS Deployment should be an integral part of a decent systems management solution. However, the technical field of OS Deployment is changing at a rapid pace. With Microsoft increasing the cadence of new version of their client/workstation/mobile whatever you want to call it this field might become either more important, or loose its importance overall. It's no secret that Microsoft is aiming for an application upgrade experience for consumers much like IOS upgrades go. In-place upgrades that sustain your data and applications is where the future lies for OS Deployment, at least to consumer devices. Whether this same approach is a good fit for businesses and enterprises remains to be seen.

It definitively poses challenges to deliver the OS Deployment service we tend to offer now-a-days. Delivering the user with a device that fully works (Is domain joined, has access to all the necessary company resources and all applications installed the user needs) is a major challenge without going through OS Deployment. I recently had a customer asking me whether he needed to wipe and load his Surface Pro 3's with the company image or whether we could use these out of the box. As with many of our customers their need for the image roughly comes down to "Make sure all the drivers are installed so there's no unknown devices in device manager, and eliminate all the vendor installed junk". Well the Surface Pro 3 image completely fits that need, so a wipe and load shouldn't be necessary. From a process perspective though that would mean the IT department would have to:  

  

1. take a machine and login with a local admin account
  

2. Join it to the domain and change the local admin password to meet with the company's needs.
  

3. Install the Configuration Manager client onto it
  

4. Wait for policies to come down
  

5. Verify whether all applications and software updates are installed
  
  
In the end, the overhead of not wiping-and-loading seemed larger than just following the standard process, as is.

So, in short, OS Deployment needs to be added to Microsoft Intune or businesses must decide that the need for OS Deployment is eliminated.

For this last bit to hold true, Microsoft needs to deliver on their failsafe OS Upgrade scenario well-enough to win the hearts of IT Departments, businesses will most likely have to adopt workplace join as opposed to domain join, and users will have to be self-sufficient to get their software themselves.

At present, and it's not because of lack of requests, Intune seems to expect that the need for OS deployment will disappear somewhere in the future, as no announcements, commitments or what-so-ever seem to indicate OS Deployment is on the roadmap anytime soon.  

### Software distribution

  
I don't think I am exaggerating when I say that in today's day and age, without Software distribution to deliver additional software, you don't have a systems management solution. People need additional software, and without local admin privileges for end users, it departments need a way to deliver. Yes, I am aware Microsoft Intune delivers software distribution as a feature. However, what they're offering out of the box isn't anything you would want to fully rely on as your main means of software distribution as it barely offers more than group policy based software distribution, and lacks flexibility and versatility.

Again, in short, Software distribution needs to be strongly enhanced or business must decide on the elimination of the need for Extended Software distribution.

For this last bit to hold true, either users need to become self-sufficient and get all their software from the store, or all applications have to become web apps most likely html5, or a mix of both.

Again when looking at the investments made into the Store, and the lack of investments in this field for Microsoft Intune my conclusion is Intune expects the need for Software Distribution to become less needed over time.  

### Server Management

  
A bit like the OS Deployment bit, and not because of lack of requests, but Microsoft Intune offers no server management worth that name.

So, either Server management needs to be added to Microsoft Intune, or businesses need to decide on the elimination of the need for server management. (Caught the theme yet?)

Again, that last bit might hold true in the future, assuming you believe in the idea that businesses are no longer going to have their own servers, and everything is hosted by a limited number of datacenter providers, that'll maintain and patch/test your servers for you. When looking at the current Azure RemoteApp offering, when you want to include custom apps, that last bit seems to bit something we might see in a distant future.  

### License Management

  
The License Management feature discussion is similar to the one on extended software distribution.

Some form of license management or elimination of the need for license management needs to occur.

Again, the latter could happen when all apps come from the store, or are html5 subscription based apps.  

### Where it the Systems Management market heading for?

  

#### Data protection

  
Data protection is key. The recent Sony hack emphasizes this point once more, data protection is critically important. In fact one might argue that this might actually be the answer to managing BYOD type of devices, where we could decide to no longer manage these devices, but start managing and protecting the data. In a world where the users can self-service most of its tasks themselves, the largest need for systems management is to  

  

1. Keep the device operational
  

2. Protect the data on the device
  
  
When the data is protected by let's say, rights management and multi-factor authentication the largest need for systems management comes down to 1. Which could be solved by allowing factory-reset like functionality.  

#### New version of Configuration Manager coming

  
A new version of Configuration Manager is coming, and it'll have a 10-year support lifecycle as all Microsoft enterprise products, so ConfigMgr Administrators are still good for a while.  

## Summary

  
Is Intune going to be replace System Center Configuration Manager, it might, but it won't happen overnight. My current point of view is that Microsoft is focusing on Microsoft Intune for mobile device management, yet have no desire to kill off a billion dollar market in managing non-mobile devices. Even if the amount of growth feasible in non-mobile device management is minimal, investments are still made. Some of them in fields were growth can be achieved (Mac & Linux mgmt anyone?). When a popular train of thought sees the light of day, and we all stop working on "Open Devices" and make the switch to "Closed aka mobile devices", that's when Microsoft is ready to pull the plug on System Center Configuration Manager, as long as that is not the case, they'll happily cash in on that billion dollar business and try to grow a multi-billion dollar cloud service right alongside it.

How I reached that point of view, is what I tried to explain in this article.

