---
title: "Modern driver management in Configuration Manager Current Branch"
author: Kim Oppalfens
header:
  overlay_image: drivers1280x960.jpg
  teaser: drivers.jpg
date: 2017-06-10
categories:
  - SCCM
  - OSD
tags:
  - SCCM
  - Drivers
---

** ###Updated after MMS 2017 session ###**
There’s a multitude of ways to handle driver management in Configuration Manager and OSDeployment.

# Intro
This post introduces a new way of handling drivers that me and my colleague Tom Degreef have started implementing at our customers with the release of Configuration Manager Current branch.

We delivered our new methodology first at a popular MMS 2017 session, based on that I am updating this blogpost where I first introduced some of the concepts back in 2016.

# Driver Management desires
The new way of doing driver management in Configuration Manager is more dynamic than it used to be. It offers the following advantages:

- Full control over the drivers that get installed
- Dynamically select these drivers based on the detected hardware model
- Install drivers during OSD without importing them into Configuration Manager
- Add a new hardware model without modifying your tasksequence in any way
- Remove static content references from the tasksequence


Intrigued yet, let's get started?
# The unseen potential of Download Package Content

The new Configuration Manager current branch has a new Tasksequence step called Download Package Content.
My MVP pall, Jörgen Nilsson, blogged about it here, http://ccmexec.com/2015/09/configmgr-vnext-feature-download-package-content/ .

On the surface, the step seems to lack the ability to do driver packages, and it doesn't appear to be Dynamic in any shape or form.
But, that's why this section's title is the Unseen potential of the step. This particular step actually has an **OVERRIDABLE** tasksequence variable called OSDDownloadDownloadPackages. This particular variable takes Package Id's as values, and then goes ahead and download those.

So we can dynamically set that variable based on the hardware model and download the relevant package to a fixed path locally on the deployed machine. Piece 1 of the puzzle is handled by a script that reads the relevant package id from an exported listed of available packages.

```posh
<#
    .Synopsis
    Short description
    .DESCRIPTION
    Long description
    .EXAMPLE
    Example of how to use this cmdlet
    .EXAMPLE
    Another example of how to use this cmdlet
#>

[CmdletBinding()]
param (
    # Param1 help description
    [Parameter(ValueFromPipelineByPropertyName,
    Position = 0)]
    [string]
    $MatchProperty = 'MifName',

    # Param1 help description
    [Parameter(ValueFromPipelineByPropertyName,
    Position = 1)]
    [string]
    $ModelName = (Get-WmiObject -Class win32_computersystemproduct -Namespace root\cimv2).Name,


    # Param2 help description
    [Parameter(ValueFromPipelineByPropertyName,
    Position = 2)]
    [string]
    $PackageXMLLibrary = ".\packages.xml",

        # Param3 help description
    [Parameter(ValueFromPipelineByPropertyName,
    Position = 3)]
    [ValidateSet("Windows 7 X64","Windows 7 X32","Windows 8 X32","Windows 8 X64","Windows 8.1 X64","Windows 8.1 X32","Windows 10 X64","Windows 10 X32","")]
    [string]
    $OSVersion = ""
)
function Get-CMCEDynamicPackage
{
  [OutputType([string])]
  Param
  (
    # Param1 help description
    [Parameter(ValueFromPipelineByPropertyName,
    Position = 0)]
    [string]
    $MatchProperty = 'MifName',

    # Param1 help description
    [Parameter(ValueFromPipelineByPropertyName,
    Position = 1)]
    [string]
    $ModelName = (Get-WmiObject -Class win32_computersystemproduct -Namespace root\cimv2).Name,


    # Param2 help description
    [Parameter(ValueFromPipelineByPropertyName,
    Position = 2)]
    [string]
    $PackageXMLLibrary = ".\packages.xml",

        # Param3 help description
    [Parameter(ValueFromPipelineByPropertyName,
    Position = 3)]
    [ValidateSet("Windows 7 X64","Windows 7 X32","Windows 8 X32","Windows 8 X64","Windows 8.1 X64","Windows 8.1 X32","Windows 10 X64","Windows 10 X32","")]
    [string]
    $OSVersion = ""
  )
  Process
  {
    #interesting properties pkgsourcepath, Description, ISVData, ISVString, Manufacturer, MifFileName, MifName, MifPublisher, MIFVersion, Name, PackageID, ShareName, Version
    [xml]$Packages = Get-Content -Path $PackageXMLLibrary

    #environment variable call for task sequence only

    try
    {
      $tsenv = New-Object -ComObject Microsoft.SMS.TSEnvironment
      $tsenvInitialized = $true
    }
    catch
    {
      Write-Host -Object 'Not executing in a tasksequence'
      $tsenvInitialized = $false
    }
    if ($OSVersion -eq "")
      {
        $PackageID = (Import-Clixml $PackageXMLLibrary | ? {$_.$MatchProperty.Split(',').Contains($ModelName)}).PackageID
        $PackageID
          if ($tsenvInitialized)
          {
            $tsenv.Value('OSDDownloadDownloadPackages') = $PackageID
          }
      else
      {
        $PackageID = (Import-Clixml $PackageXMLLibrary | ? {$_.$MatchProperty.Split(',').Contains($ModelName) -and $_.MifVersion -eq $OSVersion}).PackageID}
        $PackageID
          if ($tsenvInitialized)
          {
            $tsenv.Value('OSDDownloadDownloadPackages') = $PackageID
          }
      }
  }

}

Get-CMCEDynamicPackage -MatchProperty $MatchProperty -ModelName $ModelName -PackageXMLLibrary $PackageXMLLibrary

```

The script takes the following optional parameters:

- ModelName
- MatchProperty
- OSVersion
- PackageXMLLibrary

***Modelname*** is an optional variable that should typically be left blank. It grabs the ModelName from the Win32_computersystemproduct WMI class on the machine were the script is executed.

***MatchProperty*** is an optional variable that verifies whether the specified property contains the value of the ModelName detected. DefaultValue: MifName property of a regular Configuration Manager package.


*packageXMLLibrary* takes an optional path to the XML file you want to use, default is packages.xml in the same folder as the script.

which was generated using the following Powershell command:

```posh
Get-WmiObject -class sms_package -Namespace root\sms\site_$sitecode | Select-Object pkgsourcepath, Description, ISVData, ISVString, Manufacturer, MifFileName, MifName, MifPublisher, MIFVersion, Name, PackageID, ShareName, Version | export-clixml -path 'packages.xml' -force 

```

***OSVersion*** is an optional variable that takes one of the following values Windows 7 X64, Windows 7 X32, Windows 8 X32, Windows 8 X64, Windows 8.1 X64, Windows 8.1 X32, Windows 10 X64, Windows 10 X32

### How the script Works

The script starts with reading the XML generated with the command above, and initializing the Tasksequence COM object.


The script subsequently grabs the hardware model from WMI, it then goes through the XML to capture the package id that goes with that model.

The script expects the hardware model to be present in the MifName Field of a regular SCCM Package by default, if that doesn't suit your needs, this config can be overridden by defining your own matchproperty

Note: The above script is Powershell, so your boot image will need to contain Powershell, or you'll have to come up with something similar in VBS and CSV files

The script ends by setting the OSDDownloadDownloadPackages variable used in the Download package content step. (WistjeDatje: This tasksequence variable takes a comma seperated list of packages to download)

We are now ready to run the download package content step and dynamically download the relevant package.


Once the package is downloaded we can DISM apply it to our deployed OS image using the following command:

```posh
DISM.exe /Image:%OSDTargetSystemDrive%\ /Add-Driver /Driver:%_SMSTSMDataPath%\Drivers /Recurse /logpath:%_SMSTSLogPath%\dism.log
```

The Dism apply drivers step is a simple run command line step, without a package defined, and no filters on any of the options pages.

### Setting it all up ###
#### Create the Package Holding the script and Packages XML File
1. Create a regular Package whose data sourcepath contains the script as listed above, and note the package name down (you'll need it later on).
2. Give the script a name eg: Get-CMCEDynamicPackage.ps1 and save it to the sourcepath above
3. Run the following PowerShell Command line and copy the packages.xml to the sourcepath above

```posh
Get-WmiObject -class sms_package -Namespace root\sms\site_$sitecode | Select-Object pkgsourcepath, Description, ISVData, ISVString, Manufacturer, MifFileName, MifName, MifPublisher, MIFVersion, Name, PackageID, ShareName, Version | export-clixml -path 'packages.xml' -force 

```

#### Creating the packages for the drivers manually ####
1. Create Packages for the drivers to support the different hardware models you want to support
2. Packages should contain the extracted drivers and .Inf files

NOTE: In a follow-up post, we'll specify an easy way to create packages for new hardware as well as a way of migrating your existing driver packages to this new model.

#### Adding the tasksequence steps ####
The tasksequence itself needs 3 steps to implement the dynamic nature of applying drivers. That's the only edit needed ever. Adding additional models no longer requires editing the tasksequence.

##### Step 1.Set the PackageID of the Package holding the drivers to download
The first step can be added prior to the partitoning steps in your tasksequence. This is a Run PowerShell Script step with the following settings:
- Package: Package containing the get-CMCEDynamicPackage.PS1 script
- ScriptName: Get-CMCEDynamicPackage.PS1
- Parameters: -OSVersion 'Windows 10 X64'
- PowerShell Execution Policy: Bypass

The OS Version parameter is optional, when it's not specified the packages won't be filtered by OS Version.

![alt]({{ site.url }}{{ site.baseurl }}/images/HolyDrivers01.png)

##### Step 2. Dynamically download drivers
The second step is added after the partitioning, and before the Setup Windows and ConfigMgr step. This is a download package content step.
- Package ID: Choose a single package to download. The package you select is irrelevant, as the script in step 1 will have overridden the packageID.
- Custom Path: %_SMSTSMDataPath%\Drivers

![alt]({{ site.url }}{{ site.baseurl }}/images/HolyDrivers02.png)

##### Step 3. Dism apply the downloaded drivers
The third and last step is added after the dynamic download driver step, but before the Setup Windows and ConfigMgr step. This is a run command line step.
- Command Line: DISM.exe /Image:%OSDTargetSystemDrive%\ /Add-Driver /Driver:%_SMSTSMDataPath%\Drivers /Recurse /logpath:%_SMSTSLogPath%\dism.log

All other parameters including the package are undefined.

![alt]({{ site.url }}{{ site.baseurl }}/images/HolyDrivers03.png)

On the Options tab of this step, add error code 50 as a success code.
Success code 50 = DISM trying to install an unsigned driver while this is not allowed when not specifying the /forceunsigned parameter. The step shouldn't fail because your package might contain unsigned drivers, that you don’t want to install.

![alt]({{ site.url }}{{ site.baseurl }}/images/HolyDrivers04.png)

#### Automatically generate the XML usin status filter rules ####
Status filter rules are an incredible SCCM feature that don't get the love they deserve. They're incredible in automating SCCM stuff event-driven though. For this particular blog, we'll create 3 status filter rules to automate the management of the Packages.XML  we mentioned earlier.

There are 3 rules for the following actions:
- Package Creation
- Package Modification
- Package deletion

The script re-generateds the Packages.XML file, finds the package holding the get-CMCEDynamicPackage.PS1 script, copies the XML file there, and updates all the distribution points with the new XML.

You'll find PowerShell code to create these rules below:

**!!!!Important: The Script needs the name of the package that holds the get-cmcedynamicpackage.ps1 that you noted down earlier. 
Put that value in the $dynamicpkgname variable.!!!!!**

```posh


Import-Module "$($ENV:SMS_ADMIN_UI_PATH)\..\ConfigurationManager.psd1" # Import the ConfigurationManager.psd1 module 

$sitecode = (Get-PSDrive -PSProvider CMSite).Name
Set-Location ($sitecode + ":") # Set the current location to be the site code.

$dynamicpkgname = "get-dynamicdriverpackage"
$dynamicDriverPkgPath = ((Get-CMPackage -Name $dynamicpkgname).Pkgsourcepath)

$statusfilterrulename = "Dynamic Driver Package Insert"
New-CMStatusFilterRule -Name $statusfilterrulename -MessageType Audit -MessageId 30000 -SeverityType Informational -RunProgram $true -ProgramPath "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -command `"Get-WmiObject -class sms_package -Namespace root\sms\site_$sitecode | Select-Object pkgsourcepath, Description, ISVData, ISVString, Manufacturer, MifFileName, MifName, MifPublisher, MIFVersion, Name, PackageID, ShareName, Version | export-clixml -path '$dynamicDriverPkgPath\packages.xml' -force; (Get-WmiObject -class sms_package -Namespace root\sms\site_$sitecode | Where-Object {`$_.Name -eq '$dynamicpkgname'}).RefreshPkgSource()`""

$statusfilterrulename = "Dynamic Driver Package Update"
New-CMStatusFilterRule -Name $statusfilterrulename -MessageType Audit -MessageId 30001 -SeverityType Informational -RunProgram $true -ProgramPath "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -command `"Get-WmiObject -class sms_package -Namespace root\sms\site_$sitecode | Select-Object pkgsourcepath, Description, ISVData, ISVString, Manufacturer, MifFileName, MifName, MifPublisher, MIFVersion, Name, PackageID, ShareName, Version | export-clixml -path '$dynamicDriverPkgPath\packages.xml' -force; (Get-WmiObject -class sms_package -Namespace root\sms\site_$sitecode | Where-Object {`$_.Name -eq '$dynamicpkgname'}).RefreshPkgSource()`""

$statusfilterrulename = "Dynamic Driver Package Delete"
New-CMStatusFilterRule -Name $statusfilterrulename -MessageType Audit -MessageId 30002 -SeverityType Informational -RunProgram $true -ProgramPath "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -command `"Get-WmiObject -class sms_package -Namespace root\sms\site_$sitecode | Select-Object pkgsourcepath, Description, ISVData, ISVString, Manufacturer, MifFileName, MifName, MifPublisher, MIFVersion, Name, PackageID, ShareName, Version | export-clixml -path '$dynamicDriverPkgPath\packages.xml' -force; (Get-WmiObject -class sms_package -Namespace root\sms\site_$sitecode | Where-Object {`$_.Name -eq '$dynamicpkgname'}).RefreshPkgSource()`""

```



Enjoy.