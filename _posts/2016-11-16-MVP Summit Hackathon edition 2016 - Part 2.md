---
title: "MVP Summit hackathon edition 2016-Part 2"
author: Kim Oppalfens
related: true
date: 2016-11-20
categories:
  - SCCM
  - Configmgr
tags:
  - Hackathon
---

This years MVP summit took place a little bit over a week ago, and I believe I've recovered from jet-lag.

# Intro #
A fun and amazing part of that conference, that was introduced last year, and repeated this year, is the Hackathon. You can read about the first part of my reporting on this amazing event here: [http://www.oscc.be/sccm/configmgr/MVP-Summit-Hackathon-edition-2016-Part-1/](http://www.oscc.be/sccm/configmgr/MVP-Summit-Hackathon-edition-2016-Part-1/)

This year's edition had no less than 13 different little hacks. So let's start by introducing some more.

## OMA Uri Settings for Native ConfigMgr client ##
The new modern management of Windows 10 allows the management of a Windows 10 device through an open standard for Mobile device management called OMA. [https://en.wikipedia.org/wiki/OMA_Device_Management](https://en.wikipedia.org/wiki/OMA_Device_Management) 

This is a practical way, and the best documented way of managing some Windows 10 settings, and the only way for a very limited set of settings. This project offers the ability to set these OMA Uri's for native ConfigMgr clients in the same way you can today with an Intune enrolled client.

As new URI's become available with every new Windows build, the idea would be to enumerate available settings by browsing to a particular Windows 10 device and enumerate available settings through the OMA-WMI Bridge.

This project didn't have it's actual demo ready and could only present a UI mock-up, which most likely influenced the voting process at the end of the Hackathon.

**Related uservoice items**: 
[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/16615498-allow-defining-custom-oma-uri-settings-for-windows](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/16615498-allow-defining-custom-oma-uri-settings-for-windows) (58 votes)

## Tasksequence troubleshooting / debugger ##
This one is another Hackathon by the Genesis team and it's - on his way to absolute stardom - team member Kerwin. As we've witnessed before this team tends to add multiple eggs in it's Hackathon basket, and this year was no different.

The first portion of this years entrance into the competition was the ability to retry the phase of a deployment that can result in you ending up in Windows PE and the MP reporting that no tasksequence is available to you, or some of the referenced content is not found on any of the DP's available to you. This was elegantly implemented with a previous button in the wizard which would trigger a new policy request and CLR validation.

**Related uservoice items**:
[https://configurationmanager.uservoice.com/forums/300492-ideas/category/110049-operating-system-deployment](https://configurationmanager.uservoice.com/forums/300492-ideas/category/110049-operating-system-deployment) (135 votes)

[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/13880778-allow-retry-after-early-failures-in-osd-instead-of](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/13880778-allow-retry-after-early-failures-in-osd-instead-of) (36 votes)

A second portion was a Tasksequence debugger where you could set breakpoints, retry a step skip a step and move back in the tasksequence to a given step. This should reduce the time spent in creating a new Tasksequence and testing new steps as you no longer have to run through a complete tasksequence to evaluate a series of new actions. You control the steps from an admin UI running remotely on the Site Server.

**Related uservoice items**:
[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/13880796-f8-troubleshooting-environment-steroids-f8tes](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/13880796-f8-troubleshooting-environment-steroids-f8tes) (95 votes)

[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/13329036-allow-us-the-ability-to-view-all-variables-and-the](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/13329036-allow-us-the-ability-to-view-all-variables-and-the)
 (161 votes)

## Logo's without borders ##
This team focused on functionality over slide-ware so didn't have a PowerPoint presentation. What they did have is working Code. This feature was linked to using the new Software Center, and as the team was quickly done implementing this request, they threw in a bonus to zip up and send the client log to a central share directly from the Software Center app.
As a side note, if you see the total lack of taste of some SCCM Admins in choosing logo's we might need some sort of filter on the logo selection as a new user voice item. 

**Related uservoice items**:
[https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/15814174-separate-defining-company-logo-from-intune-subscri](https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/15814174-separate-defining-company-logo-from-intune-subscri) (14 votes)

## Asset intelligence ##
This team, "the Asset Intelligence agency", gave a great show, which is hard to bring across in a blog unfortunately.

The team wanted better Asset Intelligence, and claimed they took on the big task of not making a highly used feature better, but make people love a more or less unused feature.
The team's goals are big:
- Better defaults
- Better Admin UI
- Better data in AI!
- Leading to better insights

This lead to a project that contained a huge number of small changes.
**Related uservoice items**: Seems like the team is right, I couldn't find any related uservoice items that collected more than 5 votes.

## Better / Easier NDES Setup ##
This team had Kaido on their team, the MVP on the team that's largely known for wanting to ditch the Admin UI and replace it with just PowerShell. So, it isn't a surprise that the initial result returned into a PowerShell driven solution.

This project wants you to setup NDES much in the same way you setup a distribution point, by allowing the opportunity to have the site role setup install all prerequisites along with the site role installation. 

## Content download peek ##
For those of you that are unaware, the 1610 build of Configuration Manager adds modifications to how boundaries and boundary groups work. The client peer cache feature introduced in 1610 additionally complicates troubleshooting content related problems. To alleviate that pain, this team set forward on a way to offer better insight in content related issues.

The solution they came up with is a UI extension linked to a device that shows the Boundary groups a client is a member of, couple with the ability to select a deployment. Selecting the deployment would result in displaying the available "Distribution points" in the order the client would use them. Note that I put quotes around distribution points to stress the new Client Peer sources would show up in this interface as well.

To allow this feature to work, a client needs to update it's boundary group details almost immediately. The current thinking is to use the fast notification channel to achieve this. It's interesting to observe that there are a number of items in SCCM start to be near-realtime.

**Related uservoice items**:Another interesting observation is that Hackathon's on occasion include a solution for problems that are impacted by new features.

Be on the lookout for part 3 to conclude this blog post series. 
For those of you that haven't caught on to the theme yet, I'll spill it out for you.
If you're not submitting items to Uservoice [https://configurationmanager.uservoice.com/forums/300492-ideas](https://configurationmanager.uservoice.com/forums/300492-ideas), or at the very mimimum are using your votes to influence order of importance,
You're missing out!!!
 
 