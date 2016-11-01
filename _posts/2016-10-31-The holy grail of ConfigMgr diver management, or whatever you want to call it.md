---
title: "The holy grail of ConfigMgr driver managemement, or whatever you'd like to call it."
author: Kim Oppalfens
date: 2016-10-31
categories:
  - SCCM
  - OSD
tags:
  - SCCM
  - Drivers
---


There’s a multitude of ways to handle driver management in Configuration Manager and OSDeployment.

# Intro
Well, I don't think, this is going to be news to people that have followed my blogs over the years. When I started blogging, the plan was to tie things together and do new stuff, that hadn't been blogged before. Typically trying to solve things for which I felt was a need.
Given the title, you can expect another one of these posts.
I've never written a blog post that could have had so many different titles though:

* The best thing since sliced bread for driver management
* Why I love download package content in ConfigMgr Current Branch
* The case of the undocumented OSDDownloadDownloadPackages tasksequence variable

# Driver Management desires

One of the common desires in driver management is to have full control over which drivers get applied during your deployment.
Another desire is to install this set of drivers dynamically based on the hardware model deployed.
A third secondary desire is to do all this without actually importing drivers, this comes from Johan's method in 2007 where he introduced "Total Control" without importing drivers. http://deploymentresearch.com/Research/Post/325/MDT-2013-Lite-Touch-Driver-Management and re-created in 2012 by Achim https://www.windows-noob.com/forums/topic/10856-sccm-2012-osd-apply-driver-packages-without-importing-them-to-the-database/
And then my personal desire is to do all that without having to touch my tasksequence(s) each time a new model is introduced at one of my customers.
That's why a certain individual filled a User voice request for it, that has 496 votes at present. https://configurationmanager.uservoice.com/forums/300492-ideas/suggestions/10099479-allow-for-dynamically-selecting-apply-driver-packa , which puts it in the top 10 of currently listed requests.

In summarization we'll try and create a method that:

- Full controll over the drivers that get installed
- Dynamically select these drivers based on the detected hardware model
- Install drivers during OSD without importing them into Configuration Manager
- Add a new hardware model without modifying your tasksequence in any way


Intrigued yet, let's get started?
# The unseen potential of Download Package Content

The new Configuration Manager current branch has a new Powershell Tasksequence step called Download Package Content.
My MVP pall, Jörgen Nilsson, blogged about it here, http://ccmexec.com/2015/09/configmgr-vnext-feature-download-package-content/, don't mind the comment about someone calling it useless, that's going to be rather funny by the end of this post.
Now, I hear you think, how is that going to bring us any closer to that sliced bread for driver management? Well, this new step allows one to download a fixed list of regular packages.
On the surface, it seems to lack the ability to do driver packages, and it doesn't appear to be Dynamic in any shape or form.
But, that's why this section's title is the Unseen potential of the step. This particular step actually has an **OVERRIDABLE** tasksequence variable called OSDDownloadDownloadPackages, Yes, that's 2 times the word download in one variable. This particular variable takes PackageId's as values, and then goes ahead and download those. And lo and behold, yes, that does include driver packages. (I'll let you in on another secret, it even takes packageid's of applications).
So we can dynamically set that variable based on the hardware model and download the relevant driver package to a fixed path locally on the deployed machine. Piece 1 of the puzzle is handled by this script that reads the relevant packageid from an exported listed of available driver packages.

```posh
[xml]$Packages = get-content driverpackages.xml
#environment variable call for task sequence only
$tsenv = New-Object -COMObject Microsoft.SMS.TSEnvironment

$Model = (Get-CimInstance -ClassName win32_computersystemproduct -Namespace root\cimv2).Name

$ns = New-Object Xml.XmlNamespaceManager $Packages.NameTable
$ns.AddNamespace( "def", "http://schemas.microsoft.com/powershell/2004/04" )
$Xpathqry="/def:Objs/def:Obj//def:MS[contains(.,`"$model`")]"
$Package = ($Packages.SelectSingleNode($xpathqry,$ns))

If ($Package.SelectNodes('def:S[contains(@N,"Name")]',$ns).'#Text' -eq $Model)
{
$tsenv.Value('OSDDownloadDownloadPackages') = $Package.SelectNodes('def:S[contains(@N,"PackageID")]',$ns).'#Text'
$Package.SelectNodes('def:S[contains(@N,"PackageID")]',$ns).'#Text'
}
```

The script takes a driverpackages XML which was generated using the following powershell command:

```posh
Get-WmiObject -class sms_driverpackage -Namespace root\sms\site_poc | Select-Object Name,PackageID | export-clixml driverpackages.xml
```

It starts with reading the xml generated with the command above, and initializes the Tasksequence COM object.


The script subesequently grabs the hardware model from wmi, and subsequently  goes through the XML to capture the packageid that goes with that model. In this particular exercise, I just named my driver package after the different hardware models it applied to.
You could put the model field in the manufacturer comment or any other field of a package you desire, but you'll have to modify the script above to match that information

The next 4 lines perform an XPath query to get the package node that contains the driver package for which the name corresponds to the hardware model detected.

Note: The above script is Powershell, so your boot image will need to contain powershell, or you'll have to come up with something similar in VBS and CSV files

Note2: The parsed xml from export-clixml actually contains a Namespace, the script above shows some Powershell Xpath queries when a $ns namespace is involved
After finding the necessary packageID it's value is set in the variable 'OSDDownloadDownloadPackages'

We are now ready to run the download package content step and dynamically download the relevant driver package.
Dynamically download driver packages, check.
Once the driver package is downloaded we can Dism apply it to our deployed OS image using the following command:
DISM.exe /Image:%osdisk%\ /Add-Driver /Driver:c:\drivers\ /Recurse
Another piece of the puzzle, full control of the drivers applied by using dism, check.
The end result, a holy grail, if you will

The Dism apply drivers step is a simple run command line step, without a package defined, and no filters on any of the options pages.
The command line contains the command line referenced above:
DISM.exe /Image:%osdisk%\ /Add-Driver /Driver:c:\drivers\ /Recurse
And there you have it, no need to modify my tasksequence for a new model, check

Enjoy.