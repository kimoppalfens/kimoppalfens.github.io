---
title: "Modern end-user communication on a WDAC enforced device."
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2022-02-09
categories:
  - MEM
  - WDAC
tags:
  - SCCM
  - ConfigMgr
  - Intune
  - MEM
  - WDAC
---

We've done quite a bit of Application Control work in the past months/years. One item that is always challenging in these types of projects is end-user communication. This post details one option for improving that communication.

# The Intro #
There's a couple of items that I often see neglected in projects involving "modern". 1) Learning from past mistakes, but equally important the belief that you're supposed to treat your end users differently. Specifically, don't treat them as of they are illitereate :). Here we've set out to include users and user communication as good as we could in our WDAC project.

# The challenge #
The message you receive when an Application Control policy blocks code is a bit rudimentary. It doesn't provide a whole lot of info as to what just happened and isn't really actionable. We wanted to change both (And I am a huge fan of Martin's toast notification script so didn't mind having an excuse to send out another toast.) The nice thing about the existing message though is that it only appears when you launch something interactively. So our goals for the end-user became the following

The Goals:
* Clear and actionable end-user communication when code is blocked.
* Limit the toast notifications to a single popup per blocked application
* Only show the toast notification when a user initiated the blocked code interactively (if the user hasn't received the standard WDAC message there's no need to add additional explanation)

# The solution's building blocks #
Our eventual solution consisted of several building blocks, summarized here, as that's the stuff you'll learn in this post
## Event viewer tasks
Event viewer tasks are scheduled tasks that get triggered when an event is written to a Windows event log. Building these is typically simple enough if you can live with the basic settings. In other words if a combination of the logname, source and EventID are sufficient for your filtering needs. We quickly came to the conclusion that achieving goals 2 and 3 would not be able to be met with these basic filters.

## Vbscript ##
This served 2 purposes, 1) if you try to run a scheduled task that executes PowerShell yet you don't want to see the PowerShell Windows flash by, than apparently goold old VBScript needs to help out the new kid on the block and wrap the PowerShell. 2) To prove that, eventhough I am getting old, I still remember some of the skills I've learned over the years, like reading VBScript.

## Passing Event paramaters to an Event viewer tasks
We're not exactly doing anything with that in the toast notification but we've toyed around a bit with it in our attempts to send out alerts when a user has tried to execute blocked code.

## Martin's toast notification script
Incredibly versatile way of communicating with users in a concise and actionable way. Used this on several different projects. There were some challenges this time around though. The script generates a number of protocol handler scripts. The handler scripts are unsigned so our #wdac enforced devices didn't have much appetite for them.

# The Solution #
## Triggers ##

What we set out to build was an **Event Viewer Task** that ran in **User context** was triggered by a **Custom event filter**, triggered a **VBScript** that in turn executed a **PowerShell script** to generate an actionable **Toast notification**

The task Triggers on an event and users a Custom Event Filter

First step, figuring out a way to filter the CodeIntegrity log for entries that blocked code that was launched interactively. A code integrity block event is written to the event log with the following characteristics:

* Logname: Microsoft-Windows-CodeIntegrity/Operational
* EventID: 3077
* Si Signing Scenario: 1 (This specifies the blocked code ran in UserMode, not Kernel mode)
* Process Name: *The name of the process that was blocked from loading a file*

The **Process name** bit is what, based on our experience, appeared to give us a way to figure out code that was launched interatively and was blocked. In the past we had identified RunTimeBroker.exe (Executables launched from Start Menu), explorer.exe, cmd.exe, powershell.exe, powershell_ise.exe, msedge.exe and iexplore.exe as initial processes for interactive code. Filtering on one of these required the following XML to define a custom event filter.
 ```xml 
 <Query Id="0" Path="Microsoft-Windows-CodeIntegrity/Operational">
    <Select Path="Microsoft-Windows-CodeIntegrity/Operational">*[System[EventID = 3077] and EventData[Data[@Name="SI Signing Scenario"] = 1]] and  *[EventData[Data[@Name="Process Name"] = '\Device\HarddiskVolume3\Windows\System32\RuntimeBroker.exe']]</Select>
  </Query>
  ```
The Query Id 0 just indicates this is the first query in a querylist, we wanted to filter on multiple process names, so we needed multiple quries. The Path refers to the logfile involved. The System and Eventdata sections make most sense if you look at the XML representation of an event in Windows Event viewer. The EventID, Process Name and SI Siging Scenario fields were explained above.

The full XML of our Event viewer task trigger is as follows and includes 7 very similar queries in a query list.

```xml
<QueryList>
  <Query Id="0" Path="Microsoft-Windows-CodeIntegrity/Operational">
    <Select Path="Microsoft-Windows-CodeIntegrity/Operational">*[System[EventID = 3077] and EventData[Data[@Name="SI Signing Scenario"] = 1]] and  *[EventData[Data[@Name="Process Name"] = '\Device\HarddiskVolume3\Windows\System32\RuntimeBroker.exe']]</Select>
  </Query>
  <Query Id="1" Path="Microsoft-Windows-CodeIntegrity/Operational">
    <Select Path="Microsoft-Windows-CodeIntegrity/Operational">*[System[EventID = 3077] and EventData[Data[@Name="SI Signing Scenario"] = 1]] and  *[EventData[Data[@Name="Process Name"] = '\Device\HarddiskVolume3\Windows\explorer.exe']]</Select>
  </Query>
  <Query Id="2" Path="Microsoft-Windows-CodeIntegrity/Operational">
    <Select Path="Microsoft-Windows-CodeIntegrity/Operational">*[System[EventID = 3077] and EventData[Data[@Name="SI Signing Scenario"] = 1]] and  *[EventData[Data[@Name="Process Name"] = '\Device\HarddiskVolume3\Windows\System32\cmd.exe']]</Select>
  </Query>
  <Query Id="3" Path="Microsoft-Windows-CodeIntegrity/Operational">
    <Select Path="Microsoft-Windows-CodeIntegrity/Operational">*[System[EventID = 3077] and EventData[Data[@Name="SI Signing Scenario"] = 1]] and  *[EventData[Data[@Name="Process Name"] = '\Device\HarddiskVolume3\Windows\System32\WindowsPowerShell\v1.0\powershell.exe']]</Select>
  </Query>
  <Query Id="4" Path="Microsoft-Windows-CodeIntegrity/Operational">
    <Select Path="Microsoft-Windows-CodeIntegrity/Operational">*[System[EventID = 3077] and EventData[Data[@Name="SI Signing Scenario"] = 1]] and  *[EventData[Data[@Name="Process Name"] = '\Device\HarddiskVolume3\Windows\System32\WindowsPowerShell\v1.0\powershell_ise.exe']]</Select>
  </Query>
  <Query Id="5" Path="Microsoft-Windows-CodeIntegrity/Operational">
    <Select Path="Microsoft-Windows-CodeIntegrity/Operational">*[System[EventID = 3077] and EventData[Data[@Name="SI Signing Scenario"] = 1]] and  *[EventData[Data[@Name="Process Name"] = '\Device\HarddiskVolume3\Program Files (x86)\Microsoft\Edge\Application\msedge.exe']]</Select>
  </Query>
  <Query Id="6" Path="Microsoft-Windows-CodeIntegrity/Operational">
    <Select Path="Microsoft-Windows-CodeIntegrity/Operational">*[System[EventID = 3077] and EventData[Data[@Name="SI Signing Scenario"] = 1]] and  *[EventData[Data[@Name="Process Name"] = '\Device\HarddiskVolume3\Program Files\Internet Explorer\iexplore.exe']]</Select>
  </Query>
</QueryList>
```

## Actions ##
The Action simply calls a VBScript for the reasons earlier mentioned. So as an action we start a program:
* Program: wscript.exe
* Arguments: C:\path\tovbs\new-wdactoastnotification.vbs

The content of the VBscript is as follows
```vbnet
Dim shell,command
command = "powershell.exe -nologo -noprofile C:\path\to\toastnotificationscript\new-toastnotificationconstrained.ps1 -config c:\path\to\toastnotificationscript\config-toast-Appblocked.xml"
Set shell = CreateObject("WScript.Shell")
shell.Run command,0
```
Nothing to fancy, a shell run of a powershell script and pointing to your toastnotification config file. If you want to pass details of the event you can pass in arguments using the following syntax $(EventRecordID).

You can manipulate the other scheduled tasks settings as you desire.

Your End result should resemble the following:
A regular event running in the user's context

![General Tab](/images/WDACEventTask01.png)



















