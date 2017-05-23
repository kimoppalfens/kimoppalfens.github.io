---
title: "Setting up an SCCM Labo using Powershell Direct - Part 4"
header:
  teaser: PowerShell512x384.png
author: Tom Degreef
date: 2017-04-05
categories:
  - SCCM
  - Configmgr
  - PowerShell
  - PowerShell Direct
  - Hyper-V
  - Task Sequence
tags:
  - SCCM
  - ConfigMgr
  - PowerShell
  - PowerShell Direct
  - Hyper-V
  - Task Sequence
---

Automating a full SCCM Labo setup using PowerShell Direct. Part 4 is the Task Sequence and a template for new servers.

# Intro #

[Part 1 - Setting up a Router VM](http://www.oscc.be/sccm/configmgr/powershell/powershell%20direct/Setting-up-an-SCCM-Labo-using-Powershell-Direct-Part-1)

[Part 2 - Setting up a Domain Controller](http://www.oscc.be/sccm/configmgr/powershell/powershell%20direct/Setting-up-an-SCCM-Labo-using-Powershell-Direct-Part-2/)

[Part 3 - Setting up the Configmgr Server](http://www.oscc.be/sccm/configmgr/powershell/powershell%20direct/Setting-up-an-SCCM-Labo-using-Powershell-Direct-Part-3/)

In this final part I'll detail the outlines of the task sequence that ties all the scripts together. At the end I'll also post the script I use as a template to deploy new machines. You should be able to leverage this to get up and running in no time.


## The Task Sequence##

The Pre-requisites for the script to run successfully :

1. All the source-files that are referenced in the Powershell Direct Scripts (see previous posts)
2. Drivers for the machine you are deploying

As detailed in Part 1, we are running these labs on Intel NUC's 6th gen so obviously we have to provide the correct drivers for server 2016 (the main OS that runs the lab)

![alt]({{ site.url }}{{ site.baseurl }}/images/Task_Sequence_Part4.PNG)

## Step 1 ##

- Restart in Windows PE
- Partition Disk 0 - Bios
- Partition Disk 0 - UEFI
- Apply Operating System
- Apply Windows Settings
- Apply Network Settings
	- APPLY NUC DRIVERS
        - Allow Unsigned Drivers
        - Download Package Content
        - Dism Apply Drivers
        - Dism Apply Drivers

Really not that much spcial here. This is the beginning of any basic Configmgr OSD Task sequence.

The partitioning steps have been modified to create a 60GB OS disk and the remaining space is just a new primary partition (our data drive).

You might notice that the section for the drivers looks a bit different than the basic "Apply Driver package" step.
That's because we've used Kim's holy grail for driver management. Read up on it [here](http://www.oscc.be/sccm/osd/The-holy-grail-of-ConfigMgr-diver-management,-or-whatever-you-want-to-call-it/).


## Step 2 ##

- Download Package Content (sourcefiles)
- Inject Hyper-V into offline image

All the sourcefiles needed in any of the scripts have been grouped together in a single SCCM Package (without program). A "download package content" step is used to download all these sourcefiles locally to the harddisk of the NUC.

The location is saved to a variable called "sources" so we can re-use this location later in the task-sequence to move the files to where we want them to be.

The reason why this download is happening in WIN-PE, is to be able to use multicast when staging multiple machines. (The package is around 50GB in size).

One of the main challenges in this task sequence was to get Hyper-V up and running.
Just enabling the feature with Add-Windowsfeature,or any other way in the live OS, was failing because during the setup of Hyper-V, the server reboots twice "unattended" and this breaks the task sequence.

The solution here was break it up in 2 parts. First, we inject the Hyper-V code with DISM. This is done in the WIN-PE phase using a run commandline step :

```
cmd /c Dism.exe /image:%OSDisk%\ /Enable-Feature /FeatureName:Microsoft-Hyper-V /LogPath:%_SMSTSLogPath%\Hyper-V.log

```


## Step 3 ##

- Setup Windows and Configuration Manager
- Enable Hyper-V
- Enable Wireless
- Restart Computer
- Configure Hyper-V
- Move Setup sources to D:\Sources

The second part of getting Hyper-V up and running is again running a commandline step :

```
powershell.exe -executionpolicy bypass -command add-windowsfeature Hyper-v    -IncludeAllSubFeature -IncludeManagementTools
```

In Server 2016, if you want to use Wifi, you need to add it as a feature. The "Enable Wireless" step is a run commandline step that makes that happen :

```
powershell.exe -executionpolicy bypass -command add-windowsfeature Wireless-Networking
```

After a reboot we call a small powershell script to configure Hyper-V.

```posh
Import-Module Hyper-V

$VMDrive = “D:”
$VMPath = Join-Path -path $VMDrive -ChildPath "VM\VM"
$VHDPath = Join-Path -Path $VMDrive -ChildPath "VM\VHD"
$Sources = Join-Path -Path $VMDrive -ChildPath "Sources"

MD -Path $vmpath,$VHDPath,$Sources -ErrorAction 0

Set-VMHost -VirtualHardDiskPath $VHDPath -VirtualMachinePath $vmpath

#create virtual switches

$ethernet = Get-NetAdapter -Name ethernet
$wifi = Get-NetAdapter -Name wi-fi

New-VMSwitch -Name ExternalSwitch -NetAdapterName $ethernet.Name -AllowManagementOS $true -Notes ‘Parent OS, VMs, LAN’
New-VMSwitch -Name PrivateSwitch -SwitchType Private -Notes ‘Internal VMs only’
```

The different folders for storing the virtual machines, virtual harddisks and our source-binaries are created first.

Set-VMhost is called to configure Hyper-V in making these folders the default values when creating a new virtual machine.

Finally we create 2 virtual switches, the ExternalSwitch is used in the router virtual machine and the PrivateSwitch is used in all the other virtual machines to isolate them from the rest of the network.

All of the scripts in these blog posts were grouped together in a single package (without a program). This allows us to easily call them in a "run powershell script" step that references this package.

just enter the name of the PS1 file in the script name section and update the execution policy to bypass and you're good to go.

Once the necessary folders are created we can move our previously dowloaded "source content" (with the location captured in a variable) to the correct location so that the next scripts can reference it.

The "Move Setup sources to D:\Sources" is a run commandline step :

```
cmd.exe /c move /y  %sources01%\*.* D:\Sources
```

## Step 4 ##

- Set up the lab
    - Create Router VM
    - Create Domain Controller VM
    - Create CM VM

These 3 steps kick off the scripts created in the first 3 parts of this series.
Like the "Configure Hyper-V" script these are "Run Powershell Script" steps that call each of the scripts without any parameters and the execution policy set to "bypass".

That's it !!

Kick off the task sequence on a machine of your liking and watch your lab being setup fully automated.

Everything documented here is "work in progress" and if I make some changes that are worth mentioning I'll post an update.

I hope you enjoyed reading these posts and let me know if you managed to replicate this setup in your own environment!

Below is the template that I use to create a basic virtual machine using powershell direct.

Good luck!



## The template ##

```posh
$VMName = "MyHostName"
$Memory = 4GB
$NewVHD = "D:\VM\VHD\" +$VMName + ".vhdx"
$VHDSize = "100000000000"
$Switchexternal = "externalSwitch"
$Switchprivate = "privateSwitch"
$Cpu=2
$IP = "172.30.150.10" #set correct IP
$Localadminpwd = "MyHyper$3curePwd!"
$Domainadminpwd = "MyHyper$3cureD0ma!nPwd!"
$cminstallpwd = "U$3rPwd!"
$domainname = "MyDomain.local" 
$netbiosName = "MyDomain"

function LoopInvoke-Command ($VmName, $ScriptBlock, $Argumentlist, $Credential)
{
   $x = $false
do
{
Try
{
    
    Invoke-Command -VMName $VmName -ScriptBlock $scriptBlock -ArgumentList $Argumentlist -Credential $credential -ErrorAction Stop
    $x = $true
}
catch
{
    write-host "failed, retrying..."
    Start-Sleep -seconds 15
}
}
until ($x -eq $true) 
}


function Test-IsVMUP ($VmName, $Argumentlist, $Credential)
{
   start-sleep -Seconds 30
   $x = $false
   $scriptblock = {
param($remotesvcname)
     get-service -Name $remotesvcname
}

do
{
Try
{
    Invoke-Command -VMName $VmName -ScriptBlock $scriptBlock -ArgumentList $Argumentlist -Credential $credential -ErrorAction Stop
    $x = $true
}
catch
{
    write-host "failed vm is up, retrying..."
    Start-Sleep -seconds 2
}
}
until ($x -eq $true) 
}

Copy-Item -Path D:\Sources\2016Sysprep.vhdx -Destination $NewVHD

NEW-VM –Name $Vmname –MemoryStartupBytes $Memory -VHDPath $NewVHD -Generation 2 –Switchname $Switchprivate
SET-VMProcessor –VMName $VMName –count $Cpu 
Set-VM -VMName $VMName -DynamicMemory -MemoryMaximumBytes $memory

start-vm $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

Stop-VM –Name $vmname
start-vm $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$password = ConvertTo-SecureString $Localadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("administrator", $password )

$script = {
param($remotevmname)
rename-computer -newname $remotevmname
restart-computer
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$script = {
param($remotevmname)
$first = get-netadapter
Rename-NetAdapter -name $first.Name -NewName "Private"
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
param($remotevmname,$privateip)
New-NetIPAddress -InterfaceAlias "Private" -IPAddress $privateip -DefaultGateway 172.30.150.254 -PrefixLength 24 
Set-DnsClientServerAddress -InterfaceAlias "Private" -ServerAddresses 172.30.150.1
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$ip

start-sleep -Seconds 5

$script = {
  param($remotevmname,$domainpwd,$domain)
  $password = $domainpwd | ConvertTo-SecureString -asPlainText -Force
  $username = "$domain\administrator" 
  $credential = New-Object System.Management.Automation.PSCredential($username,$password)
  Add-Computer -DomainName $domain -Credential $credential
  restart-computer
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname, $Domainadminpwd, $domainname

$password = ConvertTo-SecureString $Domainadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("$netbiosName\administrator", $password )

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

Add-VMDvdDrive -vmname $VMName
```

I won't go into the details of this script anymore as they all have been explained in the previous blog posts.

