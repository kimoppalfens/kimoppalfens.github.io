---
title: "Set Network Access account in SCCM Current Branch using PowerShell"
header:
  overlay_image: PowerShell1280x960.png
  teaser: PowerShell512x384.png
author: Tom Degreef
date: 2017-02-22
categories:
  - SCCM
  - Configmgr
  - PowerShell
  - NAA
tags:
  - SCCM
  - ConfigMgr
  - PowerShell
  - NAA
---

Hello World ! My First blog-post on OSCC on setting a NAA in SCCM CB using PowerShell

# Intro #

As you can see in this Blogpost [Mastering Windows 10 deployment & Management in the Enterprise](http://www.oscc.be/Blog/Post/15/Win10Deploymenttraining), Kim and myself are creating a training on Windows 10 management using Configuration Manager Current Branch.

In order to provide every trainee his own lab-environment, we wanted to automate the build as much as possible. Once our setup is fully up and running I will create a few other blog posts detailing how we did it and why in a certain way <spoiler> We use a lot of PowerShell Direct </spoiler>.
One of the (many) challenges was automating the configuration of the SCCM environment in each lab. 

To be able to execute a task sequence in Configmgr I needed to create/set a "Network Access Account".

Luckily David O'Brien has done the legwork [here](https://david-obrien.net/2012/10/create-a-network-access-accountconfiguration-manager-2012/) and I was able to use most of his code.

However, it seems that after David created his post for Configmgr 2012, things have changed for the network access account (since 2012 R2 and beyond) and now you can add multiple NAA's where initially you could only add one.
So I had to update his code.

## Changes ##

1) David uses WMI to add a user-Account to the Configmgr environment and I replaced this by the "New-CMAccount" cmdlet

2) The "Software Distribution" Instance in the "SMS_SCI_ClientComp" class on the site-server no longer uses the "Props" Property but now is the "PropsLists" Property

## The Script ##

```posh
#Created : 17-02-22
#Author : Tom Degreef - OSCC

#Step 0 - Set the values for password & accountname

$pwd = ConvertTo-SecureString "MyT0pSecretP@ssword" -AsPlainText -Force
$NAA = "Domain\username"

#Step 1 - Import the Configuration Manager module                                                                
Import-Module $env:SMS_ADMIN_UI_PATH.Replace("\bin\i386","\bin\configurationmanager.psd1") -force                       
$SiteCode = Get-PSDrive -PSProvider CMSITE                                                                        
Set-Location ($SiteCode.Name +":")

#Step 2 - Create the secure password and create the SCCM User Account

New-CMAccount -Name $NAA -Password $pwd -Sitecode $SiteCode.Name

#Step 3 - make the created user account your new Network Access Account

$component = gwmi -class SMS_SCI_ClientComp -Namespace "root\sms\site_$($SiteCode.Name)"  | Where-Object {$_.ItemName -eq "Software Distribution"}

$props = $component.PropLists

$prop = $props | where {$_.PropertyListName -eq "Network Access User Names"}

$prop.Values = $NAA

$component.PropLists = $props

$component.Put() | Out-Null

```


Feel free to comment !
