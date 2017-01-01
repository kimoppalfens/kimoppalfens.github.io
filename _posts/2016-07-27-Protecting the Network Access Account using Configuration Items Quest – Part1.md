---
title: "Protecting the Network Access Account using Configuration Items Quest – Part1"
header:
  overlay_image: NetworkAccessAccount1280x960.png
  teaser: NetworkAccessAccount512x384.png
author: Kim Oppalfens
date: 2016-07-27
categories:
  - SCCM
  - Configmgr
tags:
  - Security
---


So this post is long overdue and needs to start with referencing and thanking a couple of people upon which this work is built, as happens so often in community related activities.

First, this blogpost, is an answer to a year-old blog post by my highly valued MVP-Peer Roger Zander. Roger wrote a blogpost called Network Access Account are evil which you can find here:



In that blogpost, he suggests some options to control the usage of the network access account, and refers to a technet article on how to secure the NAA. The technet article  briefly states the following Grant this account the minimum appropriate permissions on the content that the client requires to access the software. 

"_The account must have the **Access this computer from the network** right on the distribution point or other server that holds the package content."  _

_"Do not grant this account interactive logon rights" _

_"Do not grant this account the right to join computers to the domain. If you must join computers to the domain during a task sequence, use the Task Sequence Editor Domain Joining Account." _

 

When analyzing that statement it tells you what rights are needed on which servers, however, given the limited amount of permissions needed, it might not be obvious that these are **the only** permissions needed.

So I set out to grant the minimal permissions needed using Configuration Manager CI's. In these CI's I wanted to assign the User Rights needed, and further protect the account from any unneeded permissions. User rights assignment can be managed through group policy, so my quest was to stick with creating these User Right CI's in the local policy of the machines, as that is what CM typically does when items are configurable in GPO's and Configuration Manager. Anyone knowing the basics of GPO subsequently knows that the CM settings will never conflict GPO settings as these always take precedence over local policy settings.

To Summarize, I defined, and validated the approach with Jason Sandys (The Usain Bolt of the ConfigMgr MVPs @JasonSandys), the quest as:

Create Configuration Item CI's that remediate the following settings on **NON-**Distribution Points:

* Deny access this computer from the network ('SeDenyNetworkLogonRight')
* Deny logon as a batch job ('SeDenyBatchLogonRight')
* Deny logon as a service ('SeDenyServiceLogonRight')
* Deny logon locally ('SeDenyInteractiveLogonRight')
* Deny logon through Remote Desktop Services ('SeDenyRemoteInteractiveLogonRight')

This will essentially avoid any potential logins with the Network Access Account.

For Distribution points the CI's are identical with the exception that these absolutely need **Access this computer from the network** as per the Technet Article. So the effective permissions there are:
* Deny logon as a batch job
* Deny logon as a service
* Deny logon locally
* Deny logon through Remote Desktop Services

 

All of the above, serves as an introduction to the next Part, Building the CI's. Now, a lot of things can be automated, but believe you me, up until my recent discovery, automating user rights wasn't exactly part of the fun stuff in life. Now that was until Tony Pombo, decided to unlike me, stop whining about how challenging that was, but decided to fix it for us all with his Grant-UserRight powershell script. 

Hurray for Tony, and given the present candidates, "Tony Pombo for President!" Which concludes my list of Community member shoutouts in this blogpost, thanks Tony. Thanks to this Powershell script, guess I have to thank that Snover-dude as well, and the Grant-UserRight script, my quest above become as simple as running some **Get-UserRightsGrantedToAccount** and **Grant-UserRight** statements. So the meat of my different CI's are a discovery script and a remediation script.

## Discovery Script

As I split out the different items into different CI's I'll just list the most important bits of the discovery script:

if ((Get-UserRightsGrantedToAccount -Account 'domainNetworkAccessAccount').Right -eq 'SeDenyNetworkLogonRight') 

{ Write-host 'ok'}

Else

{ Write-Host 'NOT OK'}

**Remediation Script **

Grant-UserRight -Account 'domainNetworkAccessAccount' -Right SeDenyNetworkLogonRight

Enjoy.
