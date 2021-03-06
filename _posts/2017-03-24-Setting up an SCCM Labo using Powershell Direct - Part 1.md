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
  - PowerShell Direct
tags:
  - SCCM
  - ConfigMgr
  - PowerShell
  - PowerShell Direct
---

Automating a full SCCM Labo setup using PowerShell Direct. Part 1 focuses on setting up router virtual machine

# Intro #

We just finished the first iteration of our windows 10 training ([Mastering Windows 10 deployment & Management in the Enterprise](http://www.oscc.be/Blog/Post/15/Win10Deploymenttraining)) and the SCCM labs had their maiden voyage.

In the following few blogposts I will try to explain how we accomplished this automated setup and why.

Most of the work on configuring each virtual machine was done using PowerShell Direct. 

The reason we have chosen PowerShell Direct over any available hydration kit is because this allows us to deploy the lab environment(s) for training completely Zero-touch. All of the scripts I blog in these series are used in a tasksequence in our "Master" environment to completely prepare the NUCs.

## PowerShell Direct ##

You can use PowerShell Direct to run arbitrary PowerShell in a Windows 10 or Windows Server 2016 virtual machine from your Hyper-V host regardless of network configuration or remote management settings.

There are 3 ways to run PowerShell Direct: 

- As an interactive session 
- As a single-use session to execute a single command or script 
- As a persistent session


### Operating system requirements:

Host: Windows 10 or Windows Server 2016, or later running Hyper-V.

Guest/Virtual Machine: Windows 10, Windows Server Windows Server 2016, or later.

### Configuration requirements: 

- The virtual machine must be running locally on the host.
- The virtual machine must be turned on and running with at least one configured user profile.
- You must be logged into the host computer as a Hyper-V administrator.

You must supply valid user credentials for the virtual machine.

In all of the scripts I created i am using a Single-Use session to execute a bunch of steps on the hyper-v virtual machine.

## Lab Layout ##

Our SCCM Labs are powered by an Intel NUC 6th generation Core-i5 with 32GB of ram and a 480GB SSD.

As per the requirements for PowerShell Direct, the Hyper-V Host is running on Windows Server 2016.

On the Hyper-V host I created 2 virtual Switches :

- A Private Virtual switch (used to isolate all machines in the SCCM Lab from the rest of the network)
- An External switch (used to provide internet access to the SCCM Lab)
 
the internet access itself is being handled by a virtual machine doing NAT between the internal and external network. 

This blogpost is on how this "router" is setup.

The setup of the Hyper-V host itself was also automated and I'll detail that in a later post.

## The Script ##

We have a single pre-requisite for the script to run successfully :

1) A sysprepped VHDX with Server 2016 installed

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
The first part is the "configuration Part".

Here, a bunch of PowerShell variables that will be used later on throughout the script are defined for the configuration of the Virtual machine itself and the configuration of the OS.

Modifying these variables has a direct effect on the machine that it will create.

Most are self-explanatory I think, but you can fine the details on them in the next paragraph. 


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

The above piece of code is used to test if a virtual machine is up and running before continuing with the next scriptblock. I started out with start-sleeps but finding the right balance for how long to sleep became a bit of a challenge. (Thanks Kim for helping out here!)

The script takes 3 parameters :

	$Vmname is the virtual machine name that we will test

	$Argumentlist will be the service that we are testing for. If this service can be queried, we assume the machine is up. (Most of the time we use remote registry to test, but any service that starts up late in the boot process should be usable)

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

The script starts off with duplicating the sysprepped Server 2016 VHDX listed in the pre-reqs and renaming it so it has the same name as the VM it will be used in.

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

Once the machine has booted up for the very first time (and we verified that using our function), The VM gets rebooted just once more. Prior to doing this I had issues running PowerShell direct on the client.

Again, we test using our function whether the machine has booted and when it has, the first block of PowerShell direct code is run.

Basically anything in between the { } is code that will be executed locally on the client virtual machine.

Yes, this looks very very much like PowerShell remoting but the biggest difference is that I don't need network access to my virtual machine.


The first thing I do is rename the machine to a more meaningful name (that is being passed on as a parameter) and after a rename we need to reboot the machine once more.

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

The last piece of code in this script is adding a few windows features needed for Routing and Remote access to work.

Once those are installed, I configure routing and remote access using the 4 netsh commands written here above.

The unfortunate downside using netsh to configure NAT is that you can't use the routing and remote access console anymore as it complains that legacy mode is disabled.

Unfortunately I haven't been able to figure out (yet) how to accomplish this step using native PowerShell commandlets. If anybody knows .. I'm happy to update my code ;-)

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