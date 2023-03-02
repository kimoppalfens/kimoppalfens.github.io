---
title: "The year of WDAC - February 2023"
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2023-03-08
categories:
  - WDAC
tags:
  - WDAC
---

I tweeted at the start of the year that 2023 should be the year of Windows Defender Application Control aka WDAC.
The second month of the year is february, a short month, that had no less than 10 Defender for Endpoint threat analyst reports

# Intro #
This series uses Microsoft Defender for Endpoint as input on threats that surfaced in the month. More specifically we'll look at the Threat Analytics page and the Threat Campaigns that are analyzed on there. Based on the analyst reports we're doing our best to assess whether an application control implementation would have stopped this specific campaigns. 

# Methodology #
In our assessment we look at a couple of different common implementation strategies of Microsoft application control technologies. The implementations we take into consideration are Applocker setups we've seen a number of times, the WDAC implementations are a mix of implementations we've seen with possible implementations we've imagined.

In both technologies we assume your users are NOT local administrators. Implementing an application control solution when not having covered this basic measure seems counter-intuitive. 

Additionally, we look at the report and what happens prior to the hands-on-keyboard phase of a campaign, if one exist. If a hands-on-keyboard phase exists and your implementation does not block scripts and/or dll's than we assume the threat actor behind the keyboard can easily circumvent your policy by porting their tools to Powershell or dll.

# TL;DR  #
January 2023 had __10 threat campaigns__ in Microsoft Defender for Endpoint analyzed.
__2 Threat campaigns did not involve executing code on client systems__, so there's no reasonable expectation that an application control technology could stop this. We mark these campaigns as out-of-scope for our analysis and statistics.
__1 Threat campaign lacked enough details for us to make an informed decission__ as to whether an application control implementation could have stopped the campaign from wreaking havoc. We mark these campaigns as out-of-scope for our analysis and statistics.

__The 7 other campaigns would have all been stopped by any Windows Defender Application Control implementation__. 

Out of these 7, __3__ campaigns made use of dll's to evade app control implementation that do not enforce DLL rules.
__1__ campaign was PowerShell based evading application control implementations that do not enforce Script Enforcement aka PowerShell Constrained language mode.

Summary: a perfect score for the WDAC team in february, provided you don't disable script enforcement. Disabling Script enforcement would've opened you up to 1 campaign.

# The nitty gritty details #
## The campaigns ##
### Threat Insights: DEV-1101 NakedPages adversary-in-the-middle phishing kit ###
This campaign focuses on credential phishing. A technique that can't be stopped by application control technologies. This one is specifically a campaign that tries to MITM MFA authentication. As a result we score it with -1, not applicable.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-nakedpagespng.png)
### Tool profile: IceXLoader ###
This campaign kicks off from a DLL and as such bypasses application control implementations that do not enforce DLL Rules.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-icexloader.png)
### BROMINE uses custom RAT called ZORROAR for espionage ###
Fully blocked by any application control implementation that we analyse. The campaign kicks off by launching a .exe.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-Bromine.png)
### Technique profile:Threat actors abuse OneNote to deliver malware ###
This campaign kicks off from a .hta file followed by a DLL. It would bypass an application control implementation not enforcing dll's. An interesting little known fact is that WDAC already blocks .hta's when set to Audit mode. So, even if you only had WDAC enabled in audit mode this campaign would be stopped.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-Onenote.png)
### Vulnerability profile: CVE-2023-23376 Windows Common Log File System driver vulnerability ###
Fully blocked by any application control implementation that we analyse. The campaign kicks off by launching a .exe.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-CommonLogFileSystemDriver.png)
### Threat Insights: High-volume  low word count HTM/HTML phishing campaign ###
This campaign focuses on credential phishing. A technique that can't be stopped by application control technologies. As a result we score it with -1, not applicable.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-HTML phishing.png)
### Activity profile: DEV-0147 extends activity to South America ###
This campaign kicks off from a DLL and as such bypasses application control implementations that do not enforce DLL Rules.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-dev0147.png)
### CVE-2022-47966: Zoho ManageEngine unauthenticated SAML XML RCE vulnerability ###
This campaign's analyst report lacks detail for us to make a proper assessment as to whether WDAC would've stopped this.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-ManageEngine.png)
### Tool profile: Purple Fox exploit kit ###
This campaign makes uses of PowerShell exclusively, from what we can gather from the analyst report. As such, it would bypass any application control implementation that doesn't enforce scripts.
WDAC enforces script code integrity by default, but we do take up one implementation in our assessment where script enforcement is explicitely disabled. This implementation would be vulnerable to this campaign.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-PurpleFox.png)
### Attacker technique profile: Abuse of remote monitoring and management tools ###
This campaign requires admin credentials to kick off. All our application control assessments are based on users not having admin privileges. If Admin privileges have been established the threat actor could disable or manipulate your application control implementation anyway.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-RemoteTools.png)









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



