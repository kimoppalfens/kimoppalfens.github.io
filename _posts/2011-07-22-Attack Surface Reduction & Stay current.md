---
title: "Making the case for reducing (ConfigMgr) attack surface by staying current"
header:
  overlay_image: template1280x960.jpg
  teaser: template512x384.jpg
author: Kim Oppalfens
date: 2008-12-31
categories:
  - SCCM
  - Configmgr
  - AttackSurfaceReduction
  - Security
tags:
  - SCCM
  - ConfigMgr
  - AttackSurfaceReduction
  - Security
---

Header Line

# Intro #
author: Kim Oppalfens

3 years ago OSCC started offering it's knowledge of the internal working of Configuration Manager in a service to audit the security configuration of an environment. The service is called the [SCCM Vulnerability assessment](https://www.oscc.be/osccservices/Security-Audit/). Configuration Manager is an extensive product that relies on several other infrastructural systems in the Windows eco system. The issue that we've identified and talk about in this blog is one of the items we've discovered in those type of assignments. We've presented some others during multiple talks at different conferences. MMSMOA 2019 was the conference that gave Tom Degreef and me the chance to present some of these in front of a live audience.
Responsible disclosure timeline:
- Issue discovered around March / April 2019
- Issue was communicated to Microsoft early May 2019
- Microsoft confirmed the issue early June 2019
- Microsoft responed with a proposed fix and some additional questions on the 10th of June
- Configuration Manager 1906 was released including the hotfix
- October 2019, OSCC demonstrates the issue publicly for the first time at a conference.
- Today's blogpost January 27, 2021. The first release that fixed the issue has left support yesterday as can be seen [here](https://docs.microsoft.com/en-us/mem/configmgr/core/servers/manage/updates#supported-versions)

# The issue(s) #

The vulnerability revolved around 3 separate issues, which, when combined could have your entire environment compromised in no time. The 3 issues were:
1. Unauthenticated status messages and the ability to craft your own status messages
2. Status filter rule actions run as a high privileged account (LocalSystem)
3. No input validation was done prior to handing parameters over to status filter rule actions.

## Unauthenticated status messages ##
When you send a status message from a client, that status message gets threated by Configuration Manager's status message system. The status message subsystem has been around for close, if not over 2 decades by now and has been tremendously helpful in understanding what goes on in your environment. The system equally allows you to craft your own custom status messages.

[Mattias Cedervall wrote a blog](https://someguy100.wixsite.com/sccm802dot1x/post/create-a-custom-status-message-and-use-its-data-as-argument-to-a-server-side-command) about a PowerShell script to do exactly that: 

The [Microsoft SDK](https://docs.microsoft.com/en-us/mem/configmgr/develop/core/understand/configuration-manager-sdk-samples) contains a sample on how to do it 



Last, but not least, [our friends over at 2Pint](https://2pintsoftware.com/download/status-generator-manual/) released a tool that allows you to craft your own message and released it to the community, thanks Andreas!:

Given that 2Pint's tool was the only thing readily available when we started our research and in line with the saying that "No good deed goes unpunished" we went for the 2Pint tool.

## Status filter rule actions run as a high privileged account (LocalSystem)  ##
The Status Message subsystem has a way of triggering actions when a particular status message comes in. There are a number of community driven solutions out there that make use of that system. The ability to send an email when a tasksequence completes, successfully or otherwise is one popular example for how this functionality is ussed in real life. The status filter rule system responds to status messages that meet criteria you define. It equally accepts a number of items in the status message that you can pass on as variables to the action you want to execute.
A list of parameters can be found [here](https://systemscenter.ru/smsv4.en/html/4c6518c7-92b9-42a2-b94e-cc483fcac3f9.htm+&cd=2&hl=en&ct=clnk&gl=be)

The entire action configured eventually runs on the server with the privileges of the LocalSystem account.

[Matt Atkinson has a blog](http://blog.configmatt.com/2017/05/monitoring-potentially-dangerous.html) where he uses the functionality*

(*) Matt's blog was the first blog in a bing search for %msgis01

**side note** : My very first presentation at an internation conference was MMS 2010 in Las Vegas on taking SCCM event driven and contained a section on status filter rules.

## No input validation on parameters passed to status filter rule actions
When you craft your own status message you control a large number of the parameters above. The most useful one, from an abuse perspective is probably %msgis01. We used %msgis01 in combination with some continuation strings. The command prompt understands &, |, && and || as command separators. PowerShell relies on the ; character to achieve the same thing, executing multiple commands in a single string. ConfigMgr didn't validate the input of %msgis01 or any other parameter to prevent passing these characters on to the status filter rule action.

## Putting it all together. ##
Given all the knowledge gained above we could abuse status filter rules that relied on %msgis01 with the following command from 2Pint's tool.

{% highlight powershell linenos %}
.\StatGen.exe 30005 "SMS Client" -C:"Generic Component Source" -E:0 -I:" testing && c:\windows\system32\WindowsPowerShell\v1.0\powershell.exe -command new-item -itemType Directory c:\holakiekenwasmeda ","C:\mmsmoa","C:\mmsmoa" -A:400,CEN00008 -A:410,2
{% endhighlight %}

The command above does nothing harmful. It just creates a folder with the name "holakiekenwasmeda" in the root of the C drive. Checking the ownership of the folder demonstrates it was created by LocalSystem though. I'll leave the impact of having the posibility to run commands on your systems management server with a high privileged account over to the imagination of the reader. 

# The moral of the story #
This blogpost wasn't about telling you about an old vulnerability in Configuration Manager, nor was it about selling our [Configuration Manager security review](https://www.oscc.be/osccservices/Security-Audit/). It was about making the case for staying current with Configuration Manager (and other products) to reduce attack surface and stay safe. Security fixes like these aren't broadly mentioned in release notes or the What's new docs, if they are mentioned at all. That doesn't mean new builds don't contain security fixes. The issue above clearly shows, and we have it from personal experience, that Microsoft fixes reported issues. (And does so rather swiftly if the issue warrants it.)

So, If you're not updating because of the tremendous value you get in new features, with every build, you might want to consider doing it because it's the best thing to do from a security perspective. We waited with disclosing this to the general public until the vulnerable version was out of support, but a number of people have known how to do this after Microsoft released the fix.

I'd like to thank the Microsoft team involved in fixing this, the MMSMOA team for giving us an option to present about it and our Vulnerability assessment customers that gave us to freedom to investigate this and Tom Degreef the co-worker that helped build the Vulnerability assessment consulting offer.

Kim Oppalfens








