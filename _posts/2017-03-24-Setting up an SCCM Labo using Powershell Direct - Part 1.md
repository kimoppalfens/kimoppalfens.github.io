---
title: "Setting up an SCCM Labo using Powershell Direct - Part 1"
header:
  teaser: PowerShell512x384.png
author: Tom Degreef
date: 2017-03-24
categories:
  - SCCM
  - Configmgr
  - PowerShell
  - Powershell Direct
tags:
  - SCCM
  - ConfigMgr
  - PowerShell
  - Powershell Direct
---

Automating a full SCCM Labo setup using Powershell Direct.

# Intro #

We just finished the first iteration of our windows 10 training ([Mastering Windows 10 deployment & Management in the Enterprise](http://www.oscc.be/Blog/Post/15/Win10Deploymenttraining)) and the SCCM labs had their maiden voyage.

In the following few blogposts I will try to explain how we accomplished this automated setup and why.

Most of the work on configuring each virtual machine was done using Powershell Direct. 

## Powershell Direct ##

You can use PowerShell Direct to run arbitrary PowerShell in a Windows 10 or Windows Server 2016 virtual machine from your Hyper-V host regardless of network configuration or remote management settings.

There are 3 ways to run PowerShell Direct: 

As an interactive session
As a single-use session to execute a single command or script
As a persistant session

Requirements
Operating system requirements:

Host: Windows 10 or Windows Server 2016, or later running Hyper-V.
Guest/Virtual Machine: Windows 10, Windows Server Windows Server 2016, or later.

Configuration requirements: 

The virtual machine must be running locally on the host.
The virtual machine must be turned on and running with at least one configured user profile.
You must be logged into the host computer as a Hyper-V administrator.
You must supply valid user credentials for the virtual machine.

In all of the scripts I created i am singe a Single-Use session to execute a bunch of steps on the hyper-v virtual machine.

## Lab Layout ##

Our SCCM Labs are powered by an Intel NUC 6th generation Core-i5 with 32GB of ram and a 480GB SSD.

As per the requirements for Powershell Direct, the Hyper-V Host is running on Windows Server 2016.
On the Hyper-V host I created 2 virtual Switches :
 	* A Private Virtual switch (used to isolate all machines in the SCCM Lab from the rest of the network)
 	* An External switch (used to provide internet access to the SCCM Lab)
 
the internet access itself is being handled by a virtual machine doing NAT between the internal and external network. 
This blogpost is on how this "router" is setup.

The setup of the Hyper-V host itself was also automated and I'll detail that in a later post.

## The Script ##

We need a few pre-requisites for the script to run succesfully :

1) A sysprepped VHDX with Server 2016

## Step 1 ##

```posh
$VMName = "Router"
$Memory = 2GB
$NewVHD = "D:\VM\VHD\" +$VMName + ".vhdx"
$VHDSize = "100000000000"
$Switchexternal = "externalSwitch"
$Switchprivate = "privateSwitch"
$Cpu=1
$IP = "172.30.150.253" #set correct IP
$Localadminpwd = "MyHyper$3curePwd!"

```
The first part is the "configuration Part"
I created a bunch of Powershell variables that will be used later on throught the script for the configuration of the Virtual machine itself and the configuration of the OS.
Modifying these variables has a direct effect on the machine that it will create.
Most are self-explaining I think. 
$SwitchExternal & $SwitchPrivate contain the "names" of the two virtual switches created on the Hyper-V host. (see Lab Layout)
$IP is the fixed IP address the router will use on the private-part of the network
$Localadminpwd is the password to logon as a local administrator on the router

## Step 2 ##

```posh
function Test-IsVMUP ($VmName, $Argumentlist, $Credential)
{
   Start-sleep -Seconds 30
   $x = $false
   $scriptblock = 
	{
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
    	write-host "failed, retrying..."
    	Start-Sleep -seconds 2
		}
	}
until ($x -eq $true) 
}

```

The above piece of code is used to test if a virtual machine is up and running before continuing with the script. I started out with start-sleeps but this allows much more flexibility (Thanks Kim for helping out here!)
The script takes 3 parameters :
	$Vmname is the virtual machine name that we will test
	$Argumentlist will be the service that we are testing for. If this service can be queried, we assume the machine is up. (we mostly use remote registry to test but any service that starts up late in the boot process should be usable)
	$Credential will contain the securely stored credentials used to authenticate on the virtual machine

We start out with a pause of 30 seconds since it will probably take that amount of time anyway for a virtual machine to come online. If we can then reach our service the function will stop and the rest of the configuration script will continue.
If we fail on reaching that service, we will sleep for 2 seconds and then try again until we are successful.

The commandline to call the script looks like this : 

```posh
Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred
```

the $cred variable is populated using :

```posh
$password = ConvertTo-SecureString $Localadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("administrator", $password )
```

## Step 3 ##

```posh
Copy-Item -Path D:\Sources\2016Sysprep.vhdx -Destination $NewVHD

NEW-VM –Name $Vmname –MemoryStartupBytes $Memory -VHDPath $NewVHD -Generation 2 –Switchname $Switchexternal
SET-VMProcessor –VMName $VMName –count $Cpu 
SET-VM -VMName $VMName -DynamicMemory -MemoryMaximumBytes $memory
Start-VM $vmname
```

I'll start off with duplicating the sysprepped Server 2016 VHDX listed in the pre-reqs and renaming it so it has the same name as the VM it will be used in.
The next steps are for creating the virtual machine and configuring it all using the properties from Step 1
Once all configuration is done we start the Virtual machine.

## Step 4 ##

```posh
Stop-VM –Name $vmname
start-vm $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$script = {
	param($remotevmname)
	rename-computer -newname $remotevmname
	restart-computer
	}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

Once the machine has booted up for the very first time (and we verified that using our function), I'll restart this VM just once more. If I didn't do this I had issues running powershell direct on the client.
Again, we test using our function and after that it the first block of powershell direct.
Basically anthing in between the { } is code that will be executed locally on the client virtual machine.
Yes, this looks very very much like powershell remoting but the biggest difference is that I don't need network access to my virtual machine.

The first thing I do is rename the machine to a more meaningful name (that is being passed on as a parameter) and after a rename we need again to reboot the machine.

## Step 5 ##

```posh

$script = {
	param($remotevmname)
	$first = get-netadapter
	Rename-NetAdapter -name $first.Name -NewName "External"
	}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

ADD-VMNetworkAdapter –VMName $VMName –Switchname $Switchprivate 

$script = {
	param($remotevmname,$myip)
	$second = get-netadapter | ? {$_.name -ne 'External'}
	Rename-NetAdapter -name $Second.Name -NewName "Private"
	Start-Sleep -Seconds 5
	New-NetIPAddress -InterfaceAlias "Private" -IPAddress $myip -PrefixLength 24 #Private switch
	}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$IP
```

In this block of code I start off with renaming the network adapter to an easier distinguishable name as we need to add a second one for routing to work.
That's what the Add-VMNetworkAdapter step does :)
The second block of code is to rename that second adapter and configure it with the IP address configured in Step 1.

## Step 6 ##

```posh

$script = {
	param($remotevmname)
	Install-WindowsFeature Routing -IncludeManagementTools
	Install-RemoteAccess -VpnType RoutingOnly -Legacy
 
	cmd.exe /c "netsh routing ip nat install"
	cmd.exe /c "netsh routing ip nat add interface External"
	cmd.exe /c "netsh routing ip nat set interface External mode=full"
	cmd.exe /c "netsh routing ip nat add interface Private"
	}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

The last piece of code in this script is add a few windows features needed to Routing and Remote access to work
Once those are installed, I configure routing and remote access using the 4 netsh commands written here above.
The unfortunate downside using netsh to configure NAT is that you can't use the routing and remote access console anymore as it complains that legacy mode is disabled.
Unfortunately I haven't been able to figure out (yet) how to accomplish this step using native powershell commandlets. If anybody knows .. I'm happy to update my code ;-)

## Wrap up ##

You can find the full script here at the end of the blogpost.
I'll detail the setup of the domain controller in the next post but it uses similar techniques.

Feel free to comment or let me know if something isn't clear for you or isn't working out at all.

```posh

$VMName = "Router"
$Memory = 2GB
$NewVHD = "D:\VM\VHD\" +$VMName + ".vhdx"
$VHDSize = "100000000000"
$Switchexternal = "externalSwitch"
$Switchprivate = "privateSwitch"
$Cpu=1
$IP = "172.30.150.253" #set correct IP
$Localadminpwd = "MyHyper$3curePwd!"

function Test-IsVMUP ($VmName, $Argumentlist, $Credential)
{
   Start-sleep -Seconds 30
   $x = $false
   $scriptblock = 
	{
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
    	write-host "failed, retrying..."
    	Start-Sleep -seconds 2
		}
	}
until ($x -eq $true) 
}

Copy-Item -Path D:\Sources\2016Sysprep.vhdx -Destination $NewVHD

NEW-VM –Name $Vmname –MemoryStartupBytes $Memory -VHDPath $NewVHD -Generation 2 –Switchname $Switchexternal
SET-VMProcessor –VMName $VMName –count $Cpu 
Set-VM -VMName $VMName -DynamicMemory -MemoryMaximumBytes $memory
start-vm $vmname


$password = ConvertTo-SecureString $Localadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("administrator", $password )

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

Stop-VM –Name $vmname
start-vm $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

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
Rename-NetAdapter -name $first.Name -NewName "External"
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

ADD-VMNetworkAdapter –VMName $VMName –Switchname $Switchprivate 

$script = {
param($remotevmname,$myip)
$second = get-netadapter | ? {$_.name -ne 'External'}
Rename-NetAdapter -name $Second.Name -NewName "Private"
Start-Sleep -Seconds 5
New-NetIPAddress -InterfaceAlias "Private" -IPAddress $myip -PrefixLength 24 #Private switch
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$IP

$script = {
param($remotevmname)
Install-WindowsFeature Routing -IncludeManagementTools
Install-RemoteAccess -VpnType RoutingOnly -Legacy
 
cmd.exe /c "netsh routing ip nat install"
cmd.exe /c "netsh routing ip nat add interface External"
cmd.exe /c "netsh routing ip nat set interface External mode=full"
cmd.exe /c "netsh routing ip nat add interface Private"
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```