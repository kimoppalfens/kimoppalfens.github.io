---
title: "Technical Preview 1706 feature highlight : Register Windows 10 devices with Azure Active Directory"
header:
author: Tom Degreef
date: 2017-07-03
categories:
  - SCCM
  - Configmgr
  - TP
  - Intune
  - hybrid
  - AAD
  - Azure

tags:
  - SCCM
  - Configmgr
  - TP
  - Intune
  - hybrid
  - AAD
  - Azure
---

Register Windows 10 devices with Azure Active Directory explained and demoed

# Intro #

In a "normal" corporate environment, most devices are joined to your on-premise active directory. This allows you to have singe sign on to access various on-premise resources. However, with the increasing popularity of Office 365, you are most likely also accessing cloud-based resources in that same corporate environment and chances are that you need to authenticate each time you access them. 

A solution there would be to register your device also in Azure Active Directory and as such also experience SSO for cloud-based resources like the Office 365 portal or other federated Azure apps (myapps.office.com).

Additionally, when you are AAD device registered, you can also easily enable [Hello for Business](https://docs.microsoft.com/en-us/windows/access-protection/hello-for-business/hello-identity-verification) and take on the queste to get rid of passwords ;-)

In order to facilitate that Device Registration, SCCM TP 1706 has added a new feature (no challenge points this time) :

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_win10deviceregistration.PNG)

# Setting it all up #

There isn't much to set up in the first place. Navigate to the administration pane | Client Settings.
You'll find this new option under the "Cloud Services" (device) client setting.

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_DR_Clientsettings.PNG)

(it seems to be alreay enabled by default in my TP installation)

# The results & Behind the scenes #

Basically, this client setting mimics what would happen if you set the GPO to enable Device Registration (Admin Templates \ Windows Components\ Device registration enable device registration for domain joined devices).

As we all know, most of the time GPO's just modify some registry settings and here it's exactly that. Enabling that client setting modifies the following registry key : 
```
HKLM\Software\Policies\Microsoft\Windows\WorkplaceJoin - AutoWorkPlaceJoin : 1
```

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_DR_Registry.PNG)

Additionally, a scheduled task (under \Microsoft\Windows\Workplace Join) is created to perform the actual registration :

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_DR_scheduledtask.PNG)

It's the "DSRegCMD" command being executed under system context each time any user logs on.


If you have all the Pre-requisites enabled (an active Azure AD subscription and Azure AD Connect to extend the on-premises directory to Azure AD) to have device registration succeed and you run the commandline "DSREGCMD /Status" , you should see the following result after a while : 

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_DR_succeeded.PNG.PNG)

The result is that when I browse to outlook.office365.com , I don't get any prompt anymore to authenticate and thus I'm enjoying SSO to my cloud-based resources !

That's it ! Enjoy ...