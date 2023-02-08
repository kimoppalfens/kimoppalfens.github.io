---
title: "The year of WDAC - January 2023"
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2023-02-08
categories:
  - WDAC
tags:
  - WDAC
---

I tweeted at the start of the year that 2023 should be the year of Windows Defender Application Control aka WDAC.
The first month of the year aka january has flown by at an incredible pace, so let's take a look what that could've meant if you had a WDAC setup.

# Intro #
We'll use Microsoft Defender for Endpoint as our input on threats that surfaced in the month. More specifically we'll look at the Threat Analytics page and the Threat Campaigns that are analyzed on there. Based on the analyst reports we're doing our best to assess whether an application control implementation would have stopped this specific campaigns. 

# Methodology #
In our assessment we look at a couple of different common implementation strategies of Microsoft application control technologies. The implementations we take into consideration are Applocker setups we've seen a number of times, the WDAC implementations are a mix of implementations we've seen with possible implementations we've imagined.

In both technologies we assume your users are NOT local administrators. Implementing an application control solution when not having covered this basic measure seems counter-intuitive. 

Additionally, we look at the report and what happens prior to the hands-on-keyboard phase of a campaign, if one exist. If a hands-on-keyboard phase exists and your implementation does not block scripts and/or dll's than we assume the threat actor behind the keyboard can easily circumvent your policy by porting their tools to Powershell or dll.

# TL;DR  #
January 2023 had __6 threat campaigns__ in Microsoft Defender for Endpoint analyzed.
__1 Threat campaign did not involve executing code on client systems__, so there's no reasonable expectation that an application control technology could stop this. We mark these campaigns as out-of-scope for our analysis and statistics.
__2 Threat campaign lacked enough details for us to make an informed decission__ as to whether an application control implementation could have stopped the campaign from wreaking havoc. We mark these campaigns as out-of-scope for our analysis and statistics.

__The 3 other campaigns would have all been stopped by any Windows Defender Application Control implementation__. Out of these 3, __1 campaign made use of dll's, PowerShell scripts and the well-known RegSvr32 bypass for Applocker.__ The threat actors behind this campaign clearly used knowledge of Applocker and popular implementations of it to allow their campaign to move forward even when Applocker was implemented.

Summary: a perfect score for the WDAC team in january.

# The nitty gritty details #
## The campaigns ##
### Threat Insights: OAuth consent phishing trust abuse ###
This campaign focuses on credential phishing. A technique that can't be stopped by application control technologies. As a result we score it with -1, not applicable.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Jan2023-OauthConsentPhishingTrustAbuse.png)
### SystemBC tool used in human-operated ransomware intrusions
Fully blocked by any application control implementation that we analyse. The SystemBC tool kicks off by launching a .exe.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Jan2023-SystemBC tool.png)
### DEV-1039 mass SQL server exploitation continues to deliver Mallox ransomware
Fully blocked by any application control implementation that we analyse. The campaing is kicked off by a PowerShell command that launches a .exe.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Jan2023-mass SQL server exploitation.png)
### CVE-2022-47966: Zoho ManageEngine unauthenticated SAML XML RCE vulnerability
This campaign's analyst report and the vendor's report do not contain enough details to assess application control's effectiveness. 
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Jan2023-ZohoManageEngineUnauthenticatedSaml.png)
### Emerging threat group DEV-0671 deploys Cuba ransomware
This one is definitely debatable. WDAC blocks this campaign. Applocker might or might not. The initial steps would be blocked, but the threat actors clearly know how to circumvent applocker implementations.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Jan2023-CubaRansomware.png)
### DEV-0300 ransomware activity
This campaign's analyst report does not contain enough details to assess application control's effectiveness.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Jan2023-Dev-0300.png)


## Implementation Details ##
The different implementations we take into consoderation are:

* Default Applocker Exe RuleSet
  This is an implementation that starts of with the Default executable rules. It does not have the DLL Rules, Script rules or Windows Installer rules enforced. 
* Default Applocker Exe DllRuleSet
  This is an implementation that starts of with the Default executable rules and DLL Rules. It does not have the Script rules or Windows Installer rules enforced. 
* Default Applocker Exe Script RuleSet
  This is an implementation that starts of with the Default executable rules, script rules and Windows Installer rules. It does not have the DLL rules enforced.
* Default Applocker Exe Script Dll RuleSet
This is an implementation that starts of with the Default executable rules, DLL rules, script rules and Windows Installer rules. 
* Default WDAC
This is an implementation starting with the Default Wdac policy enforced. A default WDAC always enforces DLL's, Scripts, Windows Installers, which is somewhat different from Applocker.
* Default WDAC And Applocker PathRules
This is an implementation starting with the Default Wdac policy enforced, but with path rules for the Windows and Program files folders in line with default applocker rules.
* Default WDAC And ProgramFiles PathRules
This is an implementation starting with the Default Wdac policy enforced, but with path rules for the Program files folders. It does not have a Windows path rules, most things in the Windows folder are already trusted by the default policy.
* Default WDAC Without Script Enforcement
This is an implementation starting with the Default Wdac policy enforced, but has script enforcement disabled using rule option 11 Disabled: Script enforcement.
* Default WDAC Applocker EXE Dll Mimicked
This is the default WDAC policy, with script enforcement disabled, and the path rules that you see often in Applocker for Windows and program files enabled. It mimicks the Default Applocker Exe DLL ruleset in WDAC.



