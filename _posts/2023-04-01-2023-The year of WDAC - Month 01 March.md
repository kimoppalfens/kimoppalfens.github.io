---
title: "The year of WDAC - March 2023"
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2023-03-01
categories:
  - WDAC
tags:
  - WDAC
---

I tweeted at the start of the year that 2023 should be the year of Windows Defender Application Control aka WDAC.
The third month of the year is march, 3 days more than the previous month. No less than 8 additional threat analyst reports were posted to the Defender for Endpoint portal delivering a total of 18 reports.

# Intro #
This series uses Microsoft Defender for Endpoint as input on threats that surfaced in the month. More specifically we'll look at the Threat Analytics page and the Threat Campaigns that are analyzed on there. Based on the analyst reports we're doing our best to assess whether an application control implementation would have stopped this specific campaigns. 

# Methodology #
In our assessment we look at a couple of different common implementation strategies of Microsoft application control technologies. The implementations we take into consideration are Applocker setups we've seen a number of times, the WDAC implementations are a mix of implementations we've seen with possible implementations we've imagined.

In both technologies we assume your users are NOT local administrators. Implementing an application control solution when not having covered this basic measure seems counter-intuitive. 

Additionally, we look at the report and what happens prior to the hands-on-keyboard phase of a campaign, if one exist. If a hands-on-keyboard phase exists and your implementation does not block scripts and/or dll's than we assume the threat actor behind the keyboard can easily circumvent your policy by porting their tools to Powershell or dll.

# TL;DR  #
March 2023 was a busy month for the thread analysts and had no less than __18 threat campaigns__ in Microsoft Defender for Endpoint analyzed.
__3 Threat campaigns did not involve executing code on client systems__, so there's no reasonable expectation that an application control technology could stop this. We mark these campaigns as out-of-scope for our analysis and statistics.
__1 Threat campaign lacked enough details for us to make an informed decission__ as to whether an application control implementation could have stopped the campaign from wreaking havoc. We mark these campaigns as out-of-scope for our analysis and statistics.
__1 Threat campaign was around Distributed Denial of service attacks__, another item you shouldn't expect application control to stop for you.
__1 Threat campaing was actually a campaing but a description of an Exchange CVE__, there's no real way for us to know how a CVE would eventually be abused, so we do not provide an assessment for these.

__The 12 other campaigns would have all been stopped by any Windows Defender Application Control implementation enforcing the integrity of scripts__. 

Out of these 12, __5__ campaigns made use of dll's to evade app control implementation that do not enforce DLL rules.
__4__ campaigns were script based, evading application control implementations that do not enforce Script Enforcement aka PowerShell Constrained language mode.

Summary: a perfect score for the WDAC team in march, provided you don't disable script enforcement. Disabling Script enforcement would've opened you up to 5 campaigns.

# The nitty gritty details #
## The campaigns ##
### Activity profile: 3CXDesktopApp possible supply chain compromise ###
This campaign kicks off from a trojanized DLL and as such bypasses application control implementations that do not enforce DLL Rules. If your policy was build from a non-trojanized version this campaign would not be successful. 
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-feb2023-3CXDesktopApp.png)
### Activity profile: DEV-0501 shifts to BlackCat ransomware after Hive shut-down ###
Blackcat is typically delivered as a DLL. As such blocking DLL's would stop this threat.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Dev-0501Blackcat.png)
### Tool profile: Information stealers ###
We don't have enough data in this report to make our assessments so we marked it as not applicable.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-InformationStealers.png)
### Tool profile: WinDealer ###
The DLL used would definitely be blocked, it is somewhat unclear whether there is an executable involved. To side on the error of caution we're marking this as requiring DLL enforcement to be blocked.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-WinDealer.png)
### Actor profile: DEV-0506 ###
This follows qakbot infections that are blocked if you block scripts or dll's.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Dev-0506.png)
### Activity profile: PHOSPHORUS exploits Aspera Faspex vulnerability (CVE-2022-47986) ###
PowerShell based, so stopped if you enforced PowerShell Constrained language mode.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Phosphorus.png)
### CVE-2023-23397: Microsoft Outlook elevation of privilege vulnerability leads to NTLM credential theft ###
This is a credential phising campaign that does not involve code execution on the client. Not something App allow listing can solve. Marked with -1 not applicable.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Outlook Elevation of privilege.png)
### Activity profile: Emotet uses new defense evasion technique  March 2023 ###
Emoted is delivered as a DLL, to evade app allowlisting only enforcing exe's.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Emotet Evasion.png)
### Tool profile: Caffeine phishing as a service platform ###
This is a credential phising campaign that does not involve code execution on the client. Not something App allow listing can solve. Marked with -1 not applicable.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Caffeine phishing.png)
### Threat Insights: DEV-0381 incorporates SmartScreen bypasses in Magniber ransomware attacks ###
A smartscreen bypass does not equate an app allowlisting bypass. Effectively blocked by app allowlisting.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Dev0381Smartscreen.png)
### Threat Insights: Exchange vulnerability CVE-2023-21707 ###
This is not a campaign, just describing the vulnerability without details how exploits would work. As such marked as not applicable/ not possible to make an assessment.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Exchange Vulnerability.png)
### Activity profile: Remcos payload delivery through tax document lures ###
DLL's and exe's are used in that campaign exe's are blocked by all implementations.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-Remcos Payload.png)
### Threat Insights: DEV-1101 NakedPages adversary-in-the-middle phishing kit ###
This is a credential phising campaign that does not involve code execution on the client. Not something App allow listing can solve. Marked with -1 not applicable.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Mar2023-NakedPages.png)
### Activity profile: Identity focus on Qakbot attacks ###
Qakbot is known to use a mix of dll's and PowerShell scripts so blocking either would stop this.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Mar2023-Identity focus on qakbot.png)
### Behavior profile: ImageLoad from hidden directory ###
This campaign cleverly hides information from exe's on USB. Nice try, but untrusted exe's are blocked by all implementations we analyze.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Mar2023-Imageload from hidden directory.png)
### Actor profile: DEV-1010 ###
Actor uses several executables that would be blocked by an app allowlisting implementation.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Mar2023 Dev1010.png)
### DEV-0450 and DEV-0464: Distributing Qakbot for ransomware deployment ###
Qakbot is known to use a mix of dll's and PowerShell scripts so blocking either would stop this.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-Mar2023-Distributing qakbot)
### Activity profile: 2022 DDoS attack trends ###
This is a distriubted Denial of Service attack that does not involve code execution on a client. You should not expect an app allowlisting solution to solve this for you. Marked with -1 not applicable.
![alt]({{ site.url }}{{ site.baseurl }}/images/Wdac-mar2023-DDoS attack trends.png)





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



