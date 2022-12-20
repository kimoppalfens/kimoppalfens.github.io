---
title: "WDAC for the win #73 - The one with Nvidia"
header:
  teaser: mustread-512-384.jpg
author: Kim Oppalfens
date: 2022-03-09
categories:
  - WDAC
tags:
  - WDAC
  - ConfigMgr
  - Intune
  - MEM
---
s
This post details howto implement a Wdac policy to block the stolen Nvidia certs.

# Intro #

Early in March a Twitter conversation happened between a number of people regarding the Nvidia security incident that involved the leakage of some of Nvidia's expired codesigning certificates. This evolved into a recommendation by David Weston to use #WDAC as a first line of defense. You can find the twitter conversation below.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">WDAC policies work on both 10-11 with no hardware requirements down to the home SKU despite some FUD misinformation i have seen so it should be your first choice.  Create a policy with the Wizard and then add a deny rule or allow specific versions of Nvidia if you need. </p>&mdash; David Weston (DWIZZZLE) (@dwizzzleMSFT) <a href="https://twitter.com/dwizzzleMSFT/status/1499527802382471188">March 4, 2022</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## The basics ##

Creating a deny rule is something that can definitely be done in Application Control. To do that though, as for most signature based rules you need something that's called a TBS Hash. (If I am not mistaken, TBS stands for To Be Signed and it's a hash based representation of the codesigning cert.) 

The annoying bit is that you need either a file signed with the certificate or the certificate itself to be able to create that rule. That's why it took me a while to build the rule. I first had to get access to the certificate somehow.

## Building the Policy ##
I received a copy of one of the certs today, so decided to build the policy and share it out. I created the policy using the following PowerShell command.
This adds a Deny Signer for the Certificate Specified for the Windows Signing scenario, stopping code in the user space signed with this certificate.
The rule has been added to the DefaultWindows policy in audit mode.

 ~~~ powershell
Add-SignerRule -FilePath .\DefaultWindows_Audit.xml -Deny -CertificatePath .\nvidia.cer -User
Add-SignerRule -FilePath .\DefaultWindows_Audit.xml -Deny -CertificatePath .\nvidia.cer -Kernel
~~~
Some of the relevant portions of the policy are below.
The first item is a Signer for Nvidia that has the TBS value that we mentioned before.
~~~xml
    <Signer ID="ID_SIGNER_S_1_1_0" Name="NVIDIA Corporation">
      <CertRoot Type="TBS" Value="15C37DBEBE6FCC77108E3D7AD982676D3D5E77F7" />
    </Signer>
~~~

The Signer ID is than added as a Denied Signer.
~~~xml
        <DeniedSigners>
          <DeniedSigner SignerId="ID_SIGNER_S_1_1_0" />
        </DeniedSigners>
~~~

## The full policy ##
The full policy XML can be found below.

**Disclaimer**
This policy has received zero testing so far.

~~~xml
<?xml version="1.0" encoding="utf-8"?>
<SiPolicy xmlns="urn:schemas-microsoft-com:sipolicy">
  <VersionEx>10.0.0.0</VersionEx>
  <PlatformID>{2E07F7E4-194C-4D20-B7C9-6F44A6C5A234}</PlatformID>
  <Rules>
    <Rule>
      <Option>Enabled:Unsigned System Integrity Policy</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Audit Mode</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Advanced Boot Options Menu</Option>
    </Rule>
    <Rule>
      <Option>Enabled:UMCI</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Inherit Default Policy</Option>
    </Rule>
    <Rule>
      <Option>Enabled:Update Policy No Reboot</Option>
    </Rule>
  </Rules>
  <!--EKUS-->
  <EKUs>
    <EKU ID="ID_EKU_WINDOWS" Value="010A2B0601040182370A0306" FriendlyName="" />
    <EKU ID="ID_EKU_ELAM" Value="010A2B0601040182373D0401" FriendlyName="" />
    <EKU ID="ID_EKU_HAL_EXT" Value="010a2b0601040182373d0501" FriendlyName="" />
    <EKU ID="ID_EKU_WHQL" Value="010A2B0601040182370A0305" FriendlyName="" />
    <EKU ID="ID_EKU_STORE" Value="010a2b0601040182374c0301" FriendlyName="Windows Store EKU - 1.3.6.1.4.1.311.76.3.1 Windows Store" />
    <EKU ID="ID_EKU_RT_EXT" Value="010a2b0601040182370a0315" FriendlyName="" />
    <EKU ID="ID_EKU_DCODEGEN" Value="010A2B0601040182374C0501" FriendlyName="Dynamic Code Generation EKU - 1.3.6.1.4.1.311.76.5.1" />
    <EKU ID="ID_EKU_AM" Value="010a2b0601040182374c0b01" FriendlyName="AntiMalware EKU -1.3.6.1.4.1.311.76.11.1 " />
  </EKUs>
  <!--File Rules-->
  <FileRules />
  <!--Signers-->
  <Signers>
    <Signer ID="ID_SIGNER_WINDOWS_PRODUCTION_0_0_0_0" Name="Microsoft Product Root 2010 Windows EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_WINDOWS" />
    </Signer>
    <Signer ID="ID_SIGNER_ELAM_PRODUCTION_0_0_0_0" Name="Microsoft Product Root 2010 ELAM EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_ELAM" />
    </Signer>
    <Signer ID="ID_SIGNER_HAL_PRODUCTION_0_0_0_0" Name="Microsoft Product Root 2010 HAL EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_HAL_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_SHA2_0_0_0_0" Name="Microsoft Product Root 2010 WHQL EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_SHA1_0_0_0_0" Name="Microsoft Product Root WHQL EKU SHA1">
      <CertRoot Type="Wellknown" Value="05" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_MD5_0_0_0_0" Name="Microsoft Product Root WHQL EKU MD5">
      <CertRoot Type="Wellknown" Value="04" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WINDOWS_FLIGHT_ROOT_0_0_0_0" Name="Microsoft Flighting Root 2014 Windows EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_WINDOWS" />
    </Signer>
    <Signer ID="ID_SIGNER_ELAM_FLIGHT_0_0_0_0" Name="Microsoft Flighting Root 2014 ELAM EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_ELAM" />
    </Signer>
    <Signer ID="ID_SIGNER_HAL_FLIGHT_0_0_0_0" Name="Microsoft Flighting Root 2014 HAL EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_HAL_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_FLIGHT_SHA2_0_0_0_0" Name="Microsoft Flighting Root 2014 WHQL EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_TEST2010_0_0_0_0" Name="MincryptKnownRootMicrosoftTestRoot2010">
      <CertRoot Type="Wellknown" Value="0A" />
    </Signer>
    <Signer ID="ID_SIGNER_WINDOWS_PRODUCTION_USER_0_0_0_0" Name="Microsoft Product Root 2010 Windows EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_WINDOWS" />
    </Signer>
    <Signer ID="ID_SIGNER_ELAM_PRODUCTION_USER_0_0_0_0" Name="Microsoft Product Root 2010 ELAM EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_ELAM" />
    </Signer>
    <Signer ID="ID_SIGNER_HAL_PRODUCTION_USER_0_0_0_0" Name="Microsoft Product Root 2010 HAL EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_HAL_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_SHA2_USER_0_0_0_0" Name="Microsoft Product Root 2010 WHQL EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_SHA1_USER_0_0_0_0" Name="Microsoft Product Root WHQL EKU SHA1">
      <CertRoot Type="Wellknown" Value="05" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_MD5_USER_0_0_0_0" Name="Microsoft Product Root WHQL EKU MD5">
      <CertRoot Type="Wellknown" Value="04" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_WINDOWS_FLIGHT_ROOT_USER_0_0_0_0" Name="Microsoft Flighting Root 2014 Windows EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_WINDOWS" />
    </Signer>
    <Signer ID="ID_SIGNER_ELAM_FLIGHT_USER_0_0_0_0" Name="Microsoft Flighting Root 2014 ELAM EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_ELAM" />
    </Signer>
    <Signer ID="ID_SIGNER_HAL_FLIGHT_USER_0_0_0_0" Name="Microsoft Flighting Root 2014 HAL EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_HAL_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_WHQL_FLIGHT_SHA2_USER_0_0_0_0" Name="Microsoft Flighting Root 2014 WHQL EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_WHQL" />
    </Signer>
    <Signer ID="ID_SIGNER_STORE_0_0_0_0" Name="Microsoft MarketPlace PCA 2011">
      <CertRoot Type="TBS" Value="FC9EDE3DCCA09186B2D3BF9B738A2050CB1A554DA2DCADB55F3F72EE17721378" />
      <CertEKU ID="ID_EKU_STORE" />
    </Signer>
    <Signer ID="ID_SIGNER_RT_PRODUCTION_0_0_0_0" Name="Microsoft Product Root 2010 RT EKU">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_RT_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_DRM_0_0_0_0" Name="MincryptKnownRootMicrosoftDMDRoot2005">
      <CertRoot Type="Wellknown" Value="0C" />
    </Signer>
    <Signer ID="ID_SIGNER_DCODEGEN_0_0_0_0" Name="MincryptKnownRootMicrosoftProductRoot2010">
      <CertRoot Type="Wellknown" Value="06" />
      <CertEKU ID="ID_EKU_DCODEGEN" />
    </Signer>
    <Signer ID="ID_SIGNER_AM_0_0_0_0" Name="MincryptKnownRootMicrosoftStandardRoot2011">
      <CertRoot Type="Wellknown" Value="07" />
      <CertEKU ID="ID_EKU_AM" />
    </Signer>
    <Signer ID="ID_SIGNER_RT_FLIGHT_0_0_0_0" Name="Microsoft Flighting Root 2014 RT EKU">
      <CertRoot Type="Wellknown" Value="0E" />
      <CertEKU ID="ID_EKU_RT_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_RT_STANDARD_0_0_0_0" Name="Microsoft Standard Root 2001 RT EUK">
      <CertRoot Type="Wellknown" Value="07" />
      <CertEKU ID="ID_EKU_RT_EXT" />
    </Signer>
    <Signer ID="ID_SIGNER_TEST2010_USER_0_0_0_0" Name="MincryptKnownRootMicrosoftTestRoot2010">
      <CertRoot Type="Wellknown" Value="0A" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1_1_0_0_0" Name="NVIDIA Corporation">
      <CertRoot Type="TBS" Value="15C37DBEBE6FCC77108E3D7AD982676D3D5E77F7" />
    </Signer>
    <Signer ID="ID_SIGNER_S_1_1" Name="NVIDIA Corporation">
      <CertRoot Type="TBS" Value="15C37DBEBE6FCC77108E3D7AD982676D3D5E77F7" />
    </Signer>
  </Signers>
  <!--Driver Signing Scenarios-->
  <SigningScenarios>
    <SigningScenario Value="131" ID="ID_SIGNINGSCENARIO_DRIVERS_1" FriendlyName="Auto generated policy on 03-10-2022">
      <ProductSigners>
        <AllowedSigners>
          <AllowedSigner SignerId="ID_SIGNER_WINDOWS_PRODUCTION_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_ELAM_PRODUCTION_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_HAL_PRODUCTION_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_SHA2_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_SHA1_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_MD5_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WINDOWS_FLIGHT_ROOT_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_ELAM_FLIGHT_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_HAL_FLIGHT_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_FLIGHT_SHA2_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_TEST2010_0_0_0_0" />
        </AllowedSigners>
        <DeniedSigners>
          <DeniedSigner SignerId="ID_SIGNER_S_1_1" />
        </DeniedSigners>
      </ProductSigners>
    </SigningScenario>
    <SigningScenario Value="12" ID="ID_SIGNINGSCENARIO_WINDOWS" FriendlyName="Auto generated policy on 03-10-2022">
      <ProductSigners>
        <AllowedSigners>
          <AllowedSigner SignerId="ID_SIGNER_WINDOWS_PRODUCTION_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_ELAM_PRODUCTION_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_HAL_PRODUCTION_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_SHA2_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_SHA1_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_MD5_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WINDOWS_FLIGHT_ROOT_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_ELAM_FLIGHT_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_HAL_FLIGHT_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_WHQL_FLIGHT_SHA2_USER_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_STORE_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_RT_PRODUCTION_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_DRM_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_DCODEGEN_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_AM_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_RT_FLIGHT_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_RT_STANDARD_0_0_0_0" />
          <AllowedSigner SignerId="ID_SIGNER_TEST2010_USER_0_0_0_0" />
        </AllowedSigners>
        <DeniedSigners>
          <DeniedSigner SignerId="ID_SIGNER_S_1_1_0_0_0" />
        </DeniedSigners>
      </ProductSigners>
    </SigningScenario>
  </SigningScenarios>
  <UpdatePolicySigners />
  <CiSigners>
    <CiSigner SignerId="ID_SIGNER_STORE_0_0_0_0" />
    <CiSigner SignerId="ID_SIGNER_S_1_1_0_0_0" />
  </CiSigners>
  <HvciOptions>0</HvciOptions>
  <BasePolicyID>{A244370E-44C9-4C06-B551-F6016E563076}</BasePolicyID>
  <PolicyID>{A244370E-44C9-4C06-B551-F6016E563076}</PolicyID>
  <Settings>
    <Setting Provider="PolicyInfo" Key="Information" ValueName="Name">
      <Value>
        <String>DefaultWindowsAudit</String>
      </Value>
    </Setting>
    <Setting Provider="PolicyInfo" Key="Information" ValueName="Id">
      <Value>
        <String>031017</String>
      </Value>
    </Setting>
  </Settings>
</SiPolicy>
~~~








