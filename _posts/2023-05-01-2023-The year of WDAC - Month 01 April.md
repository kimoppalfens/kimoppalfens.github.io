---
title: "The year of WDAC - April 2023"
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2023-03-01
categories:
  - WDAC
tags:
  - WDAC
---

# Intro #
This series uses Microsoft Defender for Endpoint as input on threats that surfaced in the month. More specifically we'll look at the Threat Analytics page and the Threat Campaigns that are analyzed on there. Based on the analyst reports we're doing our best to assess whether an application control implementation would have stopped this specific campaigns. 

# TL;DR  #
The threat analysts at Microsoft delivered __14__ threat reports in April 2023. 
No less than __4 campaigns__ made use of code in __DLL's__ to execute their malicious code. Continuing the trend to abuse this as a way to execute code. A trend that bypasses application control implementations that do NOT take __DLL's__ into account.

__1 campaign__ relied on PowerShell scripting to do its bidding.

The PaperCut vulnerability downloads its malicious binary to a subfolder in c:\Windows and would bypass Applocker implementations that make use of the Default Rules to make the Windows OS binaries trusted.

We didn't analyze 2 out of the 14 campaigns, 1 didn't have enough data for us to make an assessment, the other one targeted IOS and as such is considered to be out of scope.

Summary: a perfect score for the WDAC team in march, provided you don't disable script enforcement. Disabling Script enforcement would've opened you up to 1 campaign.


# Methodology #
In our assessment we look at a couple of different common implementation strategies of Microsoft application control technologies. The implementations we take into consideration are Applocker setups we've seen a number of times, the WDAC implementations are a mix of implementations we've seen with possible implementations we've imagined.

In both technologies we assume your users are NOT local administrators. Implementing an application control solution when not having covered this basic measure seems counter-intuitive. 

Additionally, we look at the report and what happens prior to the hands-on-keyboard phase of a campaign, if one exist. If a hands-on-keyboard phase exists and your implementation does not block scripts and/or dll's than we assume the threat actor behind the keyboard can easily circumvent your policy by porting their tools to Powershell or dll.



# The nitty gritty details #
## The campaigns ##
### Actor profile: Storm-1219 ###
This actor typically makes use of standard remote access tools, which exe's should be stopped by your Application Control implementation.

### Activity profile: Lace Tempest exploits critical PaperCut MF vulnerabilities ###
This activity makes use of vulnerability in PaperCut that allows code execution as NT Authority\System. As such, when code execution is achieved using the DLL the code then possesses the necessary rights to disable Application Control technologies.
The detected activity used PowerShell to download the DLL, but that PowerShell code would proably be able to run in Constrained language mode. Only DLL blocking would block this activity. on top of that this activity downloads its binary to the c:\windows folder and in doing so __abuses the default Applocker rules__ that trust everything in C:\windows.

### Activity profile: Storm-1232 delivers malware through compromised United States tax filing service ###
This method uses a stolen codesigning cert to deliver an executable. Application Control should block this unless you have a publisher rule for the owner of the CodeSigning certificate.

### Behavior profile: DLL sideloading and DLL search order hijacking ###
Not a real campaign, more a reminder of a technique being used by Malware authors. As the name implies anything blocking DLL's would thrump this technique.

### Threat Insights: Nation-state threat actor Mint Sandstorm refines tradecraft to attack high-value targets ###
Makes use of Python code and Impacket, code that should not be allowed to run if you haven't made the Python interpreter trusted.
### MERCURY and DEV-1084: Destructive attack on hybrid environment ###
This attack started of using a high privileged account. Our assumption is that you protect your high privilege accounts. This is not something Application Control can prevent easily as new rules could be created by the high-privilege account owner.

### Actor profile: DEV-1118 ###
Makes use of Python code and Impacket, code that should not be allowed to run if you haven't made the Python interpreter trusted.

### Tool Profile: Qakbot ###
To the best of our knowledge, Qakbot still relies on a mix of DLL code and PowerShell code that would be blocked by an Application Control implementation that blocks untrusted DLL's and enforces Constrained Language mode for untrusted PowerShell.

### Threat insights: DEV-0196: Quadreams KingsPawn malware used to target civil society in Europe  North America  the Middle East  and Southeast Asia ###
This campaign targeted devices running IOS. As such it can not be reasonabilly be expected to be stopped by a Windows Application Control technology. Marked with -1 not applicable.

### Activity profile: DEV-0950 exploits GoAnywhere to deliver Truebot and Clop ransomware ###
Another campaign making use of DLL based code, continuing the trend that an Application Control implementation that does NOT look at DLL's can be bypassed.

### Vulnerability profile: CVE-2023-28252 Windows Common Log File System driver vulnerability ###
The Common Log File System Driver runs as LocalSystem. As a result any exploit against this driver grants the malware author the rights to create new rules and/or disable your Application Control implementation. In short, allowing any form of unvalidated code execution (exe, dll or script) can serve as a bypass to your policy. The Common Log File System driver is becoming a more popular target as it's exploitation delivers very similar opportunities to exploiting the Print Spooler service.

### Actor profile: STRONTIUM ###
This report lacks the technical detail for us to make an assessment. Marked with -1 not applicable.

### Activity profile: 3CXDesktopApp possible supply chain compromise ###
DLL's are used in that campaign, in contrast with the report last month (march 2023), there's no longer mention of an Exe, other than the 3CXDesktop app exe itself that would be blocked. Another indication that using DLL's to circumvent Application Control policies that do not take this into account are circumvented ever more frequently.

### Tool profile: BlackCat ransomware ###
To the best of our knowledge the BlackCat ransomware is typically deleivered as a .exe file. All Application Control implementations should block this exe.

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



