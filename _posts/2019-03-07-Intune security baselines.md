---
title: "Intune - Security baselines (Preview)"
header:
author: Tom Degreef
date: 2000-01-11
categories:
  - Intune

tags:
  - Intune
  - GPO
  - Security Baseline
  - Modern Management
  - Cloud
  - Azure
---

The modern alternative for security baseline GPO's

# Intro #

As a consultant, I haven't come across a single company that doesn't use any GPO's. Actually, it seems more like most company's have so many GPO's that they are afraid to touch them because it might break something... sounds familiar ? 
If you are a cloud-only company with no on-premise infrastructure, it is pretty difficult to apply gpo-like settings only through Intune. Yes, you could create custom CSP's or work with powershell scripts, but creating CSP's isn't the most straightforward thing to do and using powershell scripts to replace what GPO's do feels a bit like re-inventing the wheel. 

With every feature-release of Windows 10 (latest iteration when this article is published is Windows 10-1809), Microsoft also releases a set of security baselines.  

## What are security baselines ##

Every organization faces security threats. However, the types of security threats that are of most concern to one organization can be completely different from another organization. For example, an e-commerce company may focus on protecting its Internet-facing web apps, while a hospital may focus on protecting confidential patient information. The one thing that all organizations have in common is a need to keep their apps and devices secure. These devices must be compliant with the security standards (or security baselines) defined by the organization. 
A security baseline is a group of Microsoft-recommended configuration settings that explains their security impact. These settings are based on feedback from Microsoft security engineering teams, product groups, partners, and customers. 

**note:** The above information is taken from the [Microsoft documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-security-baselines)

## Why are security baselines needed ##

Security baselines are an essential benefit to customers because they bring together expert knowledge from Microsoft, partners, and customers. 
For example, there are over 3,000 Group Policy settings for Windows 10, which does not include over 1,800 Internet Explorer 11 settings. Of these 4,800 settings, only some are security-related. Although Microsoft provides extensive guidance on different security features, exploring each one can take a long time. You would have to determine the security impact of each setting on your own. Then, you would still need to determine the appropriate value for each setting. 
In modern organizations, the security threat landscape is constantly evolving, and IT pros and policy-makers must keep up with security threats and make required changes to Windows security settings to help mitigate these threats. To enable faster deployments and make managing Windows easier, Microsoft provides customers with security baselines that are available in consumable formats, such as Group Policy Objects backups.

**note:** The above information is taken from the [Microsoft documentation](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-security-baselines)

But, until recently, these security baselines were only available as GPO's. However, with the release of Intune 1901, Microsoft has now provided us with the same functionality in Intune

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline01.png)

**note:** The security baselines available through Intune do not match 100% with their GPO counterpart. There are more GPO settings available than there are Intune settings. I will try to cook up a different blogpost covering the differences.

Apart from those differences, some of the settings available through Intune could use a bit more polishing in my option. Some section names, settings and descriptions are not as clear as they could be and could lead to some confusion. This blogpost is dedicated to try and take some of that confusion away :)

# The Settings #

Upon clicking the Security Baseline (Preview) link in the Intune management UI, you can select the baselines for the October 2018 build of Windows 10 (Windows 10 - 1809)

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline02.png)

Selecting that allows you to create a profile and manage the settings. This blogpost won't cover that in detail as there are plenty of blogs out there that will guide you through that scenario. 
Please be warned that these baselines are still a preview. If you want to start using them, make sure to test on a very small set of machines first so you fully understand the impact of the settings and be aware that those settings might change before they remove the "preview"-tag.

As of now, we have about 30 sections available in the settings part of the profile. If we take a closer look at the first one 

## Block Display of toast notifications ##

Intune :
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline03.png)

GPO alternative :

Policy Setting name : Turn off toast notifications on the lock screen
Setting : Enabled
Help Text : "This policy setting turns off toast notifications on the lock screen.
        If you enable this policy setting, applications will not be able to raise toast notifications on the lock screen.
        If you disable or do not configure this policy setting, toast notifications on the lock screen are enabled and can be turned off by the administrator or user.
        No reboots or service restarts are required for this policy setting to take effect."

As you can see, the Intune team changed the name and help text for this particular setting. I assume the goal here is to make it more clear for the Intune administrators as to what this setting is about.
I personally have no preference for the intune or the GPO version, so both are equally good I would say.

Let's take the next example 

## Block game DVR(desktop only) ##

Intune :
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline04.png)

GPO alternative :

Policy Setting name : Enables or disables Windows Game Recording and Broadcasting
Setting : Disabled
Help Text : "Windows Game Recording and Broadcasting.
This setting enables or disables the Windows Game Recording and Broadcasting features. If you disable this setting, Windows Game Recording will not be allowed.
If the setting is enabled or not configured, then Recording and Broadcasting (streaming) will be allowed."

Now for me, the GPO is rather clear, I enable or disable game recording, the setting is set to Disabled and the help text also uses the words Enable and disable. The Intune alternative is a bit more confusing for me. Why is it desktop only ? There is no mention of this in the GPO. I assume here that "desktop only" doesn't refer to the formfactor of you windows 10 device but rather that this is a Windows 10 only setting and not a mobile device setting. However, when you create a profile, the platform is set fixed to "Windows 10 and later", so you wouldn't be able to apply this setting to a mobile device.
Finally, if you read the help text for Intune : Configures whether recording and broadcasting of games is allowed. and you see the setting "Yes" it seems to indicate that this setting does allow you to record games.

My preference is the GPO in this case as it's a lot more clear to what the setting is about.

## Bitlocker ##

Intune :
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline05.png)

GPO alternative :

I won't copy over all the GPO settings that cover bitlocker as there way to many settings.  That being said, I prefer the Intune approach here. They only provide a small set of bitlocker settings versus all the GPO ones that are, in my opinion , way to difficult to handle unless you are really deep into bitlocker security 

GPO example :
Policy Setting name : Use enhanced Boot Configuration Data validation profile
Setting : no default
Help Text : "This policy setting allows you to choose specific Boot Configuration Data (BCD) settings to verify during platform validation.
If you enable this policy setting, you will be able to add additional settings, remove the default settings, or both.
If you disable this policy setting, the computer will revert to a BCD profile similar to the default BCD profile used by Windows 7.
If you do not configure this policy setting, the computer will verify the default Windows BCD settings. 
Note: When BitLocker is using Secure Boot for platform and Boot Configuration Data (BCD) integrity validation, as defined by the ""Allow Secure Boot for integrity validation"" group policy, the ""Use enhanced Boot Configuration Data validation profile"" group policy is ignored.
The setting that controls boot debugging (0x16000010) will always be validated and will have no effect if it is included in the provided fields."

For me it's clear, less is more :)

## Browser ##

Intune :
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline06.png)

In this case, it's not really the comparison with the GPO that is confusing, rather the help text in Intune itself.

Require SmartScreen for Microsoft Edge : Yes

For me, this is clear, I enable smartscreen for Edge... However if we read the helptext : 
Microsoft Edge uses Windows Defender SmartScreen (turned on) to protect users from potential phishing scams and malicious software by default. Also, by default, users cannot disable (turn off) Windows Defender SmartScreen. Enabling this policy turns off Windows Defender SmartScreen and prevent users from turning it on. Don’t configure this policy to let users choose to turn Windows defender SmartScreen on or off.

This seems to indicate that if you enable this setting, you prevent users from turning on smartscreen.

The corresponding GPO is a bit more clear 

Policy Setting name : Configure Windows Defender SmartScreen
Setting : Enable
Help Text : "This policy setting lets you configure whether to turn on Windows Defender SmartScreen. Windows Defender SmartScreen provides warning messages to help protect your employees from potential phishing scams and malicious software. By default, Windows Defender SmartScreen is turned on.
If you enable this setting, Windows Defender SmartScreen is turned on and employees can't turn it off.
If you disable this setting, Windows Defender SmartScreen is turned off and employees can't turn it on.
If you don't configure this setting, employees can choose whether to use Windows Defender SmartScreen."

## Device Guard ##

Intune :
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline07.png)

Apart from the section called Device guard and the settings covering credential guard, this is rather clear. The GPO counterpart covers both setting in 1 gpo,but I think the Intune version is a lot clearer ! (the GPO also lists this setting under device guard)

The system guard setting seems to be lacking some help text, but the GPO alternative is even worse as there is no "System Guard" setting. If you dig into the details, the Virtualization based security GPO does set some systemguard registry keys, but this is not clear from the help text or policy settings name

## Device Lock ##

Intune :
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline08.png)

The contradiction here is in the "Password minimum character set count" with a value of 3. For me, this seems to indicate that you need at least 3 characters in your password. However, the help text indicates that the numbers 1 through 4 define the complexity if your password.

I assume that the help text is correct for "Password minimum character set count" as a few lines below there is "Minimum password length" with a value of 8.
there the help text matches the setting properly.

## Event log service ##

Intune :
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline09.png)

Another "Less is more" approach that I like. It clearly states what the setting is about and leaves all the non essential settings out of the picture

## Experience ##

Intune :
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline10.png)

This one is also interesting as the Intune team here proposes you to block the Windows Spotlight feature, where the GPO team has no default configuration for these settings.
Again, I prefer the Intune approach here.

## Internet Explorer ##

We've got a lot of settings here. Depending how deep you are into Internet explorer security, it could be interesting to go over all of them. 
While I was digging into these settings and comparing them to the security GPO's, I thought I found differences between what the Intune team sets as default and what the GPO team did. However, then I noticed that the Intune settings were not grouped by zones like the GPO's :

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline11.png)

This makes it very difficult to compare both configurations unfortunately. I counted both the configured security GPO's and the Intune settings for Internet explorer (116 !) and my conclusion is that the Intune team included all GPO's that had a default value configured (I took some samples that seem to confirm this)

There are in total 836 GPO's related to internet explorer settings, so I think the Intune team made the right choice here by only exposing the ones that seem to matter most. However I would very much like that they grouped them more together based on the function of the setting.

## Local Policies Security Options ##

Restrict anonymous access to named pipes and shares

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline12.png)

If you don't read the help text, the intention of this setting is clear to me, but when you do read the help text, it gets confusing again as they refer to "Enabled" but the setting allows you to set "Yes" or "Not Configured".

Require client to always digitally sign communications

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline13.png)

At the end of the help text, there is a reference to 2 other policies when this setting is disabled. From the Intune side, you have no option to disable this policy as you can only "Enable" it (configure it to YES) or "Not Configured". Both policies are also not configurable from the Intune side.

They do exist at a GPO level and are enabled by default : 
Domain member: Digitally encrypt secure channel data (when possible)
Domain member: Digitally sign secure channel data (when possible)

Prevent clients from sending unencrypted passwords to third party SMB servers

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline14.png)

We are starting to see a pattern here. The settings is clear, if you select "Yes", clients won't be able to send unencrypted passwords to 3rd party SMB servers. However, the help text seems to indicate that if you Enable this policy , you DO allow unencrypted passwords to be sent to 3rd party SMB servers.

Require admin approval mode for administrators

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline15.png)

Interestingly enough, here we have references to "Not Configured" in the help text and not Enable/Disable anymore.

Allow remote calls to security accounts manager

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline18.png)

The confusion here is the format you have to use to restrict RPC connections to SAM. The GPO reference is identical to the Intune one and both have no explanation on the format in use. If you are interested, it is [SDDL (Security Descriptor Definition Language)](https://itconnect.uw.edu/wares/msinf/other-help/understanding-sddl-syntax/)

## MS Security Guide ##

None of the settings have any explanation on what they actually do. Even the "learn more" link doesn't help that much. I'll include the GPO help text here for your reference as it might help clear things up a bit 

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline19.png)

Apply UAC restrictions to local accounts on network logon

"This setting controls whether local accounts can be used for remote administration via network logon (e.g., NET USE, connecting to C$, etc.). Local accounts are at high risk for credential theft when the same account and password is configured on multiple systems.  Enabling this policy significantly reduces that risk.
Enabled (recommended): Applies UAC token-filtering to local accounts on network logons. Membership in powerful group such as Administrators is disabled and powerful privileges are removed from the resulting access token. This configures the LocalAccountTokenFilterPolicy registry value to 0. This is the default behavior for Windows.
Disabled: Allows local accounts to have full administrative rights when authenticating via network logon, by configuring the LocalAccountTokenFilterPolicy registry value to 1.
For more information about local accounts and credential theft, see ""Mitigating Pass-the-Hash (PtH) Attacks and Other Credential Theft Techniques"": http://www.microsoft.com/en-us/download/details.aspx?id=36036.
For more information about LocalAccountTokenFilterPolicy, see http://support.microsoft.com/kb/951016."

SMB v1 client driver start configuration

"Configures the SMB v1 client driver's start type.
To disable client-side processing of the SMBv1 protocol, select the ""Enabled"" radio button, then select ""Disable driver"" from the dropdown.
WARNING: DO NOT SELECT THE ""DISABLED"" RADIO BUTTON UNDER ANY CIRCUMSTANCES!
For Windows 7 and Servers 2008, 2008R2, and 2012, you must also configure the ""Configure SMB v1 client (extra setting needed for pre-Win8.1/2012R2)"" setting.
To restore default SMBv1 client-side behavior, select ""Enabled"" and choose the correct default from the dropdown:
* ""Manual start"" for Windows 7 and Windows Servers 2008, 2008R2, and 2012;
* ""Automatic start"" for Windows 8.1 and Windows Server 2012R2 and newer.
Changes to this setting require a reboot to take effect.
For more information, see https://support.microsoft.com/kb/2696547"

SMB v1 server

"Disabling this setting disables server-side processing of the SMBv1 protocol. (Recommended.)
Enabling this setting enables server-side processing of the SMBv1 protocol. (Default.)
Changes to this setting require a reboot to take effect.
For more information, see https://support.microsoft.com/kb/2696547"

Digest authentication

"When WDigest authentication is enabled, Lsass.exe retains a copy of the user's plaintext password in memory, where it can be at risk of theft. Microsoft recommends disabling WDigest authentication unless it is needed.
If this setting is not configured, WDigest authentication is disabled in Windows 8.1 and in Windows Server 2012 R2; it is enabled by default in earlier versions of Windows and Windows Server.
Update KB2871997 must first be installed to disable WDigest authentication using this setting in Windows 7, Windows 8, Windows Server 2008 R2 and Windows Server 2012.
Enabled: Enables WDigest authentication.
Disabled (recommended): Disables WDigest authentication. For this setting to work on Windows 7, Windows 8, Windows Server 2008 R2 or Windows Server 2012, KB2871997 must first be installed.
For more information, see http://support.microsoft.com/kb/2871997 and http://blogs.technet.com/b/srd/archive/2014/06/05/an-overview-of-kb2871997.aspx ."


Structured exception handling overwrite 

"If this setting is enabled, SEHOP is enforced. For more information, see https://support.microsoft.com/en-us/help/956607/how-to-enable-structured-exception-handling-overwrite-protection-sehop-in-windows-operating-systems.
If this setting is disabled or not configured, SEHOP is not enforced for 32-bit processes."

## MSS Legacy ##

Same as with MS Security Guide, we have no details on the settings. I'll include the GPO details, however unlike with the MS Security guide, the GPO help text doesn't contain that much more details.

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline20.png)

Network IP source routing protection level

MSS: (DisableIPSourceRouting) IP source routing protection level (protects against packet spoofing)

Network ignore NetBIOS name release requests except from WINS servers

MSS: (NoNameReleaseOnDemand) Allow the computer to ignore NetBIOS name release requests except from WINS servers

Network IPv6 source routing protection level

MSS: (DisableIPSourceRouting IPv6) IP source routing protection level (protects against packet spoofing)

Network ICMP redirects override OSPF generated routes

MSS: (EnableICMPRedirect) Allow ICMP redirects to override OSPF generated routes

## Power ##

There is nothing wrong with the settings , help text and values. However, it lacks consistency again with the other settings. Most settings are configurable with "Yes" or "Not Configured", but here we got drop down boxes again with "Enabled", "Disabled" and "Not configured". For now, I prefer this way as at least the help text matches the values better

## Remote Management ##

the Intune settings as such are not confusing, but it does show that Microsoft is constantly tweaking the settings text and help text.  Some time ago, "Block Storing run as credentials" had no clear description, but now it does. However, the Microsoft documentation hasn't been updated yet and still shows no real details on this setting

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline21.png)
![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline22.png)

## Smart Screen ##

Require SmartScreen for apps and files

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline23.png)

The corresponding GPO allows more control than just Enabling or "not configuring" the setting. I haven't validated yet if the Intune and GPO set the same registry value (that is what I would expect), but then there are just some options missing in the Intune version it seems.

GPO help text : 

"This policy allows you to turn Windows Defender SmartScreen on or off.  SmartScreen helps protect PCs by warning users before running potentially malicious programs downloaded from the Internet.  This warning is presented as an interstitial dialog shown before running an app that has been downloaded from the Internet and is unrecognized or known to be malicious.  No dialog is shown for apps that do not appear to be suspicious.
Some information is sent to Microsoft about files and programs run on PCs with this feature enabled.
If you enable this policy, SmartScreen will be turned on for all users.  Its behavior can be controlled by the following options:
* Warn and prevent bypass
* Warn
If you enable this policy with the ""Warn and prevent bypass"" option, SmartScreen's dialogs will not present the user with the option to disregard the warning and run the app.  SmartScreen will continue to show the warning on subsequent attempts to run the app.
If you enable this policy with the ""Warn"" option, SmartScreen's dialogs will warn the user that the app appears suspicious, but will permit the user to disregard the warning and run the app anyway.  SmartScreen will not warn the user again for that app if the user tells SmartScreen to run the app.
If you disable this policy, SmartScreen will be turned off for all users.  Users will not be warned if they try to run suspicious apps from the Internet.
If you do not configure this policy, SmartScreen will be enabled by default, but users may change their settings."

## Windows Defender ##

Although the section header seems to indicate that these settings are all about windows Defender, I found settings related to [Attack surface reduction rules](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-exploit-guard/attack-surface-reduction-exploit-guard#rule-block-all-office-applications-from-creating-child-processes) :

- Office apps launch child process type
- Script downloaded payload execution type
- Email content execution type
- Script obfuscated macro code type
- Office apps other process injection type
- Office macro code allow Win32 imports type
- Office apps executable content creation or launch type

It also contains settings for Exploit Guard :

- Network protection type

And , for me , the most confusing one is this reference to Credential guard :

![alt]({{ site.url }}{{ site.baseurl }}/images/IntuneSecurityBaseline24.png)

We already have a section on credential guard that seems to configure the same setting, but with a slightly different option.

# Conclusion # 

It seems that the Intune team wants to align the values to configure security baseline policies with the other settings in the Intune UI (Yes vs Enabled), but the help text to properly explain all the settings mostly still contain references to Enable or Disable. This is not the case everywhere as there are some good settings with good help texts.

On the other hand, this is still a preview and I have noticed that the setting or text is being changed/tweaked without notice. So chances are that these "issues" are still being fixed before they take away the "preview" tag.

The way Intune approaches the security baselines makes it easier for most admins to go through the settings and understand what they are about compared to what you need to do for the GPO security baselines. In some examples we even have a much cleared description than the GPO counterpart and the "less is more" approach is something that I also appreciate.

If this appraoch allows you to enable most/all of these security settings in your company versus the overwhelming GPO approach, then this is most certainly a step in the right direction.

