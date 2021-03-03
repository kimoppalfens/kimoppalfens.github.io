---
title: "Reducing attack surface with Application Control and Managed Installers."
tagline: "This post will explain the basics of how a Windows Defender Application Control managed installer works."
header:
  overlay_image: Wdac-Tom-1280-x960.jpg
  teaser: Wdac-Tom-512-x384.jpg
author: Kim Oppalfens
date: 2021-02-18
categories:
  - AttackSurfaceReduction
tags:
  - SCCM
  - ConfigMgr
  - Intune
  - Security
  - AttacksurfaceReduction
---


# Intro #

This is another post in the attack surface reduction series. In this installment let's start discussing application control. Application control peaked my interest a couple of years ago and that interest sky rocketted when I heard about the managed installer functionality introduced in Windows 10 1709.

Apart from the public [docs at microsoft](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/windows-defender-application-control) the goto resource on Wdac is [Matt Graeber aka @mattifestation](https://twitter.com/mattifestation?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor). I've learned a ton from his many posts on the topic.

Between Matt's posts and the docs there's quite a lot out there on Wdac itself. One thing severily lacking, other than frustrations uthered in several forums, is how you manage and maintain this in an environment that's under the control of a systems management solution. That's where managed installers come in.  I'll try to fill in some of the detail around that.

## The basics ##
Managed installers are supposed to make all software installed through your Systems Management platform trusted by your Application Control policy. Given my background in this area (I have been and still am an MVP for 15 years in this field of expertise) the platforms in scope are Configuration Manager and Intune. So everything deployed by either of those solutions would pass your Application Control policy and be allowed.
Again, working in this field the end goal has always been to install every single thing needed through that centralized platform as this eliminates the need for admin accounts to install things.

So how does this work? Whenever something is downloaded and/or installed by a managed installer, in other words whenever something is written to disk by a Managed Installer a "tag" is left on that file detailing the originating process that wrote the file to disk. The "tag" uses an NTFS feature called extended attributes to store that data. The somewhat peculiar thing about Managed Installers is that they have been implemented through a mix of Microsoft Applocker settings & Windows Defender Application Control. You define the rules for Managed Installers in Microsoft Applocker AND you specify a ruleoption in Windows Defender Application Control that you want the Managed Installers defined to pass your WDAC checks.

That's roughly all there is to it. The tagging of the files works just fine while your WDAC policy is still in audit mode. In other words, you can do your future self, or anyone following you in the sysadmin a huge favour by just enabling it now.

If you think through what I wrote above for a minute the conclusion is that Managed Installers do nothing for software and files written to disk prior to defining your Managed Installers. The files written to disk weren't tagged at the time they were written to disk and there's no way to tag them after the fact. I assume it is becoming more clear that enabling this now might be hugely beneficial down the road.

## The applocker setup ##
There's no real UI out there for building these managed installer rules so we're typically deployting a script that merges the rules we want with any Microsoft Applocker policy that exists (if any). The script equally launches and configures the necessary Microsoft Applocker services that need to be running before Applocker does its thing. Something that has bitten many admins before. Another reason for some admins to rely on scripting this is that defining Managed Installer rules through the Applocker CSP is unsupported to date. As a result Intune admins have no choice but to rely on scripting the rules.

The script :

{% highlight powershell linenos %}

param (
    [switch] $CheckComplianceOnly = $false
	)

# Variables
[System.Int32]$policyBinaryTimeoutSeconds = 300
[System.Int32]$waitBatchSeconds = 5
[System.Int32]$maxWaitSeconds = 300

[string]$miPolicyBinaryPathRoot = "$env:windir\System32"

if(-not ([Environment]::Is64BitProcess))
{
    $miPolicyBinaryPathRoot = "$env:windir\Sysnative"
}

[string]$miPolicyBinaryPath = Join-Path -Path $miPolicyBinaryPathRoot -ChildPath "AppLocker\ManagedInstaller.AppLocker"

[string]$SccmMiPolicy = 
@"
<AppLockerPolicy Version="1">
    <RuleCollection Type="Appx" EnforcementMode="NotConfigured" />
    <RuleCollection Type="Dll" EnforcementMode="AuditOnly">
        <FilePathRule Id="86f235ad-3f7b-4121-bc95-ea8bde3a5db5" Name="Dummy Rule" Description="Dummy Rule" UserOrGroupSid="S-1-1-0" Action="Deny">
            <Conditions>
                <FilePathCondition Path="%OSDRIVE%\ThisWillBeBlocked.dll" />
            </Conditions>
        </FilePathRule>
        <RuleCollectionExtensions>
            <ThresholdExtensions>
                <Services EnforcementMode="Enabled" />
            </ThresholdExtensions>
            <RedstoneExtensions>
                <SystemApps Allow="Enabled" />
            </RedstoneExtensions>
        </RuleCollectionExtensions>
    </RuleCollection>
    <RuleCollection Type="Exe" EnforcementMode="AuditOnly">
        <FilePathRule Id="9420c496-046d-45ab-bd0e-455b2649e41e" Name="Dummy Rule" Description="Dummy Rule" UserOrGroupSid="S-1-1-0" Action="Deny">
            <Conditions>
                <FilePathCondition Path="%OSDRIVE%\ThisWillBeBlocked.exe" />
            </Conditions>
        </FilePathRule>
        <RuleCollectionExtensions>
            <ThresholdExtensions>
                <Services EnforcementMode="Enabled" />
            </ThresholdExtensions>
            <RedstoneExtensions>
                <SystemApps Allow="Enabled" />
            </RedstoneExtensions>
        </RuleCollectionExtensions>
    </RuleCollection>
    <RuleCollection Type="ManagedInstaller" EnforcementMode="Enabled">
        <FilePublisherRule Id="6cc9a840-b0fd-4f86-aca7-8424a22b4b93" Name="CMM - CCMEXEC.EXE, 5.0.0.0+, Microsoft signed" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
            <Conditions>
            <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="SYSTEM CENTER CONFIGURATION MANAGER" BinaryName="CCMEXEC.EXE">
                <BinaryVersionRange LowSection="5.0.0.0" HighSection="*" />
            </FilePublisherCondition>
            </Conditions>
        </FilePublisherRule>
        <FilePublisherRule Id="780ae2d3-5047-4240-8a57-767c251cbb12" Name="CCM - CCMSETUP.EXE, 5.0.0.0+, Microsoft signed" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
            <Conditions>
            <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="SYSTEM CENTER CONFIGURATION MANAGER" BinaryName="CCMSETUP.EXE">
                <BinaryVersionRange LowSection="5.0.0.0" HighSection="*" />
            </FilePublisherCondition>
            </Conditions>
        </FilePublisherRule>
        <FilePublisherRule Id="70104ed1-5589-4f29-bb46-2692a86ec089" Name="MICROSOFT.MANAGEMENT.SERVICES.INTUNEWINDOWSAGENT.EXE version 1.38.300.1 exactly in MICROSOFT® INTUNE™ from O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
            <Conditions>
                <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="MICROSOFT® INTUNE™" BinaryName="MICROSOFT.MANAGEMENT.SERVICES.INTUNEWINDOWSAGENT.EXE">
                    <BinaryVersionRange LowSection="1.38.300.1" HighSection="*" />
                </FilePublisherCondition>
            </Conditions>
        </FilePublisherRule>
    </RuleCollection>
    <RuleCollection Type="Msi" EnforcementMode="NotConfigured" />
    <RuleCollection Type="Script" EnforcementMode="NotConfigured" />
</AppLockerPolicy>
"@

function MergeAppLockerPolicy([string]$policyXml)
{
    $policyFile = '.\AppLockerPolicy.xml'
    $policyXml | Out-File $policyFile

    Write-Host "Merging and setting AppLocker policy"

    Set-AppLockerPolicy -XmlPolicy $policyFile -Merge -ErrorAction SilentlyContinue

    Remove-Item $policyFile
}

function VerifyCompliance([xml]$policy)
{
    $result = $false
    $miNode = $policy.AppLockerPolicy.ChildNodes | Where-Object{$_.Type -eq 'ManagedInstaller'}

    if(-not $miNode)
    {
        Write-Host('Policy does not contain any managed installers')
    }
    else
    {
        $ccmexecNode = $miNode.ChildNodes | Where-Object{($_.LocalName -eq 'FilePublisherRule') -and ($_.Name -eq 'CMM - SMSWD.EXE, 5.0.0.0+, Microsoft signed')}

        if(-not $ccmexecNode)
        {
            Write-Host('Policy does not have CCMEXEC managed installer policy.')
        }
        else
        {
            $ccmsetupNode = $miNode.ChildNodes | Where-Object{($_.LocalName -eq 'FilePublisherRule') -and ($_.Name -eq 'CCM - CCMSETUP.EXE, 5.0.0.0+, Microsoft signed')}

            if(-not $ccmsetupNode)
            {
                Write-Host('Policy does not have CCMSetup managed installer policy.')
            }
            else
            {
                $result = $true
            }
        }
    }

    return $result
}

# Execution flow starts here
# Get and load the current effective AppLocker policy
try
{
    [xml]$effectivePolicyXml = Get-AppLockerPolicy -Effective -Xml -ErrorVariable ev -ErrorAction SilentlyContinue
}
catch 
{ 
    Write-Error('Get-AppLockerPolicy failed. ' + $_.Exception.Message)
    exit 10
}

# Check if it contains MI policy and if the MI policy has rules for Ccmsetup/CcmExec
try
{
    $compliant = VerifyCompliance($effectivePolicyXml)
}
catch
{
    Write-Error('Failed to verify AppLocker policy compliance. ' + $_.Exception.Message)
    exit 12
}

if($compliant)
{
    Write-Host("AppLocker policy is compliant")
    exit 0
}

Write-Host("AppLocker policy is not compliant")

if($CheckComplianceOnly)
{
    exit 2
}

# Start services
Write-Host 'Starting services'

[Diagnostics.Process]::Start("$env:windir\System32\sc.exe","start gpsvc")
[Diagnostics.Process]::Start("$env:windir\System32\appidtel.exe","start -mionly")

[System.Int32]$waitedSeconds = 0

# Check service state, wait up to 1 minute
while($waitedSeconds -lt $maxWaitSeconds)
{
    Start-Sleep -Seconds $waitBatchSeconds
    $waitedSeconds += $waitBatchSeconds

    if(-not ((Get-Service AppIDSvc).Status -eq 'Running'))
    {
        Write-Host 'AppID Service is not fully started yet.'
        continue
    }

    if(-not ((Get-Service appid).Status -eq 'Running'))
    {
        Write-Host 'AppId Driver Service is not fully started yet.'
        continue
    }

    if(-not ((Get-Service applockerfltr).Status -eq 'Running'))
    {
        Write-Host 'AppLocker Filter Driver Service is not fully started yet.'
        continue
    }

    break
}

if (-not ($waitedSeconds -lt $maxWaitSeconds))
{
    Write-Error 'Time-out on waiting for services to start.'
    exit 1
}

# Set the policy
try
{
    MergeAppLockerPolicy($SccmMiPolicy)
}
catch
{
    Write-Error('Failed to merge AppLocker policy. ' + $_.Exception.Message)
    exit 14
}

# Wait for policy update
if(test-path $miPolicyBinaryPath)
{
	$previousPolicyBinaryTimeStamp = (Get-ChildItem $miPolicyBinaryPath).LastWriteTime
	Write-Host ('There is an existing ManagedInstaller policy binary (LastWriteTime: {0})' -f $previousPolicyBinaryTimeStamp.ToString('yyyy-MM-dd 
HH:mm'))
}

if($previousPolicyBinaryTimeStamp)
{
	$action = 'updated'
	$condition = '$previousPolicyBinaryTimeStamp -lt (Get-ChildItem $miPolicyBinaryPath).LastWriteTime'
}
else
{
	$action = 'created'
	$condition = 'test-path $miPolicyBinaryPath'
}

Write-Host "Waiting for policy binary to be $action"

$startTime = get-date

while(-not (Invoke-Expression $condition))
{
	Start-Sleep -Seconds $waitBatchSeconds

	if((new-timespan $startTime $(get-date)).TotalSeconds -ge $policyBinaryTimeoutSeconds)
	{ 
		Write-Error "Policy binary has not been $action within $policyBinaryTimeoutSeconds seconds"
	    exit 1
	}
}

Write-Host ('Policy binary was created after {0:mm} minutes {0:ss} seconds' -f (new-timespan $startTime $(get-date)))

# Check compliance again
try
{
    [xml]$effectivePolicyXml = Get-AppLockerPolicy -Effective -Xml -ErrorVariable ev -ErrorAction SilentlyContinue
}
catch 
{ 
    Write-Error('Get-AppLockerPolicy failed. ' + $_.Exception.Message)
    exit 10
}

# Check if it contains MI policy and if the MI policy has rules for Ccmsetup/CcmExec
try
{
    $compliant = VerifyCompliance($effectivePolicyXml)
}
catch
{
    Write-Error('Failed to verify AppLocker policy compliance. ' + $_.Exception.Message)
    exit 12
}

if($compliant -eq $false)
{
    Write-Error("AppLocker policy is not compliant")
    exit 1
}

Write-Host 'AppLocker with Managed Installer for CcmExec/CcmSetup successfully enabled'

{% endhighlight %}







