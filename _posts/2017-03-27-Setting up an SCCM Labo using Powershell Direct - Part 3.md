---
title: "Setting up an SCCM Labo using Powershell Direct - Part 3"
header:
  teaser: PowerShell512x384.png
author: Tom Degreef
date: 2017-03-27
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

Automating a full SCCM Labo setup using PowerShell Direct. Part 3 is the setup and configuration of a Configmgr Server.

# Intro #

[Part 1 - Setting up a Router VM](http://www.oscc.be/sccm/configmgr/powershell/powershell%20direct/Setting-up-an-SCCM-Labo-using-Powershell-Direct-Part-1)

[Part 2 - Setting up a Domain Controller](http://www.oscc.be/sccm/configmgr/powershell/powershell%20direct/Setting-up-an-SCCM-Labo-using-Powershell-Direct-Part-2/)

In this part we will setup the SCCM Server including all pre-requisites for a fully unattended installation and configuration.
Just like in the previous parts the beginning of the script is identical. We duplicate the sysprepped harddisk, rename the machine, configure the network settings and join the domain.


## The Script ##

The Pre-requisites for the script to run successfully :

1. A sysprepped VHDX with Server 2016 installed
2. The SCCM 1606 ISO
3. The Server 2016 DVD (for "offline" Dotnet installation)
4. SQL Server sources (in our case SQL 2016 SP1)
5. SQL Server Management Studio (this is now a separate download)
5. "Offline" ADK setup files
6. CMUpdates (During SCCM installation you need the downloaded updates)

The Pre-reqs that don't come in an Iso like the ADK setup files, CMupdates, SSMS,... were converted manually to an iso using any tool available on the internet.
I added sometimes a few other files to those ISO's not to have to many different sources but I'll detail that once that section comes up.

## Step 1 ##

```posh
$VMName = "SCCM_Primary"
$Memory = 12GB
$NewVHD = "D:\VM\VHD\" +$VMName + ".vhdx"
$VHDSize = "100000000000"
$Switchexternal = "externalSwitch"
$Switchprivate = "privateSwitch"
$Cpu=4
$IP = "172.30.150.2"
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

NEW-VM -Name $Vmname -MemoryStartupBytes $Memory -VHDPath $NewVHD -Generation 2 -Switchname $Switchprivate
SET-VMProcessor -VMName $VMName –count $Cpu 
Set-VM -VMName $VMName -MemoryMaximumBytes $memory

start-vm $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

Stop-VM –Name $vmname
start-vm $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$password = ConvertTo-SecureString $Localadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("administrator", $password )

$NewVHD = "D:\VM\VHD\" +$VMName + "_D.vhdx"
$VHDSize = "100000000000"

New-VHD -Path $newvhd -SizeBytes $vhdsize
Add-VMHardDiskDrive -VMName $VMName -Path $newvhd

##Add extra harddisk

$script = {
  param($remotevmname)
  Get-Disk | `
  Where-Object partitionstyle -eq 'raw' | `
  Initialize-Disk -PartitionStyle MBR -PassThru | `
  New-Partition -AssignDriveLetter -UseMaximumSize | `
  Format-Volume -FileSystem NTFS -NewFileSystemLabel "Data" -Confirm:$false
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  rename-computer -newname $remotevmname
  restart-computer
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

Add-VMDvdDrive -vmname $VMName

$script = {
  param($remotevmname)
  $first = get-netadapter
  Rename-NetAdapter -name $first.Name -NewName "Private"
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname,$myip)
  New-NetIPAddress -InterfaceAlias "Private" -IPAddress $myip -DefaultGateway 172.30.150.254 -PrefixLength 24 
  Set-DnsClientServerAddress -InterfaceAlias "Private" -ServerAddresses 172.30.150.1
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$IP

$script = {
  if ((Get-DnsClientServerAddress -InterfaceAlias "Private" | ? {$_.AddressFamily -eq '2'} ).ServerAddresses[0] -ne "172.30.150.1")
  {
    Write-Error "First dns server not set correctly"
  }
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname,$domainpwd)
  $domain = "Training.Local"
  $password = $domainpwd | ConvertTo-SecureString -asPlainText -Force
  $username = "$domain\administrator" 
  $credential = New-Object System.Management.Automation.PSCredential($username,$password)
  Add-Computer -DomainName $domain -Credential $credential
  restart-computer
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname, $Domainadminpwd

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$password = ConvertTo-SecureString $Domainadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("training\administrator", $password )
```

This should look rather familiar by now, although a few things have been changed.
This virtual machine is created without the "DynamicMemory" option as this seemed to have a negative impact on the SQL setup performance.
The following section is new as well :

```posh
$NewVHD = "D:\VM\VHD\" +$VMName + "_D.vhdx"
$VHDSize = "100000000000"

New-VHD -Path $newvhd -SizeBytes $vhdsize
Add-VMHardDiskDrive -VMName $VMName -Path $newvhd

$script = {
  param($remotevmname)
  Get-Disk | `
  Where-Object partitionstyle -eq 'raw' | `
  Initialize-Disk -PartitionStyle MBR -PassThru | `
  New-Partition -AssignDriveLetter -UseMaximumSize | `
  Format-Volume -FileSystem NTFS -NewFileSystemLabel "Data" -Confirm:$false
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

I create first a 100GB (dynamic) VHDX file, add it to the virtual machine and then format it. This harddrive will be used as a sources share for any SCCM Related installation binaries.

Once the VM is domain joined we create a new credential variable for the domain administrator.

## Step 2 ##

```posh
Set-VMDvdDrive -VMname $vmname -Path D:\Sources\en_windows_server_2016_x64_dvd_9327751.iso

$script = {
  param($remotevmname)
  Install-WindowsFeature Net-Framework-Core -source e:\sources\sxs
  net localgroup administrators /add Training\CMinstall
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  Add-WindowsFeature RSAT-ADDS, RSAT-ADDS-Tools
  Import-Module ActiveDirectory
  $computer = $remotevmname + "$"
  ADD-ADGroupMember "CmServers" –members $computer
  restart-computer
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$password = ConvertTo-SecureString $Cminstallpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("training\CMInstall", $password )
```

We mount the Server 2016 Iso so we can reference it for our DotNet 3.5 installation. We add the "CMInstall" account (created during the Domain controller setup) to the local administrators group.
Finally we install the Remote Server Administration Tools for AD management so we can add ourselves to the CMServers AD group. Just like the CMInstall account,this group was also created during the Domain Controller setup.
We reboot and create a new credential variable so we can continue installation with the CMInstall account.

## Step 3 ##

```posh
Set-VMDvdDrive -VMname $vmname -Path D:\Sources\en_sql_server_2016_standard_with_service_pack_1_x64_dvd_9540929.iso

$script = {
    if (test-path "e:\setup.exe")
    { write-host "found setup.exe, launching sql setup"}
    else
    {
        write-error "e:\setup.exe not found"
    }
}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  new-item -Path C:\SQLUpdates -itemtype Directory
  e:\setup.exe /ACTION=Install /IACCEPTSQLSERVERLICENSETERMS /UpdateEnabled=1 /UpdateSource=c:\SQLUpdates /features="SQLEngine" "RS" "TOOLS" /installshareddir="c:\SQL\(X64)" /InstallsharedwowDir="c:\SQL\(X86)" /instancedir="c:\SQL\(X64)" /Instancename=MSSQLSERVER /q /AGTSVCACCOUNT="NT Authority\System" /INSTALLSQLDATADIR="c:\SQL\(X64)" /SQLBACKUPDIR="c:\MSSQLSERVER\BACKUP" /SQLCOLLATION=SQL_Latin1_General_CP1_CI_AS /SQLSYSADMINACCOUNTS="Training\domain admins" "Primary\administrator" "Training\cminstall" /SQLSVCACCOUNT="NT Authority\System" /SQLTEMPDBDIR="c:\MSSQLSERVER\TEMPDB" /SQLTEMPDBLOGDIR="c:\MSSQLSERVER\LOGS" /SQLUSERDBDIR="c:\MSSQLSERVER\USERDB" /SQLUSERDBLOGDIR="c:\MSSQLSERVER\Logs"  /RSSVCACCOUNT="NT AUTHORITY\SYSTEM" /INDICATEPROGRESS 
  start-sleep -seconds 30
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
    if ((Get-Process | ? {$_.Name -like "*setup*" }).count -eq 0)
    {
        write-host "setup.exe not found, assuming sql setup finished"
    }
    else 
    {Write-Error "setup.exe found, sql setup still running"}


}
loopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Set-VMDvdDrive -VMname $vmname -Path D:\Sources\SSMS.iso

$script = {
  param($remotevmname)
  e:\SSMS-Setup-ENU /Install /quiet /norestart
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
    start-sleep -Seconds 3
    if ((Get-Process | ? {$_.Name -eq "SSMS-Setup-ENU"}).count -eq 0)
    {
        write-host "setup.exe not found, assuming sql setup finished"
    }
    else 
    {Write-Error "setup.exe found, sql setup still running"}


}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

The SQL DVD is mounted and we test that we can find setup.exe using our loopinvoke-command function. 
For SQL setup to run successfully we also need a folder where SQL may download updated setup files to.

```posh
new-item -Path C:\SQLUpdates -itemtype Directory
```

SQL Setup is launched with the SQL Engine installation, Reporting server and Tools.
The easiest for creating the setup commandline is running SQL Setup manually using the parameters that you need in your environment. After the configuration wizard is finished, an INI file is created with the full unattended setup command.

As long as we see setup running (with our loopinvoke-command function) we halt the script.

After that, we mount our own created iso with the Management studio setup and launch it. As usual, we halt further execution of the script by monitoring the existance of , in this case, the "SSMS-Setup-ENU" process.

## Step 4 ##

```posh
Set-VMDvdDrive -VMname $vmname -Path D:\Sources\ADK1607.iso

$script = {
  param($remotevmname)
  add-windowsfeature rdc
  add-windowsfeature UpdateServices-API
  Add-windowsfeature BITS, BITS-IIS-EXT, RSAT-BITS-SERVER
  Add-windowsfeature WEB-MGMT-Compat, WEB-WMI
  New-NetFirewallRule -DisplayName "SQL Server" -Direction Inbound –Protocol TCP –LocalPort 1433 -Action allow
  New-NetFirewallRule -DisplayName "SQL Admin Connection" -Direction Inbound –Protocol TCP –LocalPort 1434 -Action allow
  New-NetFirewallRule -DisplayName "SQL Database Management" -Direction Inbound –Protocol UDP –LocalPort 1434 -Action allow
  New-NetFirewallRule -DisplayName "SQL Service Broker" -Direction Inbound –Protocol TCP –LocalPort 4022 -Action allow
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  e:\adksetup.exe /log c:\cmupdates\adksetup.log /Features OptionId.DeploymentTools OptionId.UserStateMigrationTool OptionId.WindowsPreinstallationEnvironment /norestart /ceip off /quiet
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
    if ((Get-Process | ? {$_.Name -eq "adksetup.exe" -or $_.Name -like "*msiexec*"}).count -eq 0)
    {
        write-host "adksetup.exe not found, assuming adk setup finished"
    }
    else 
    {Write-Error "adksetup.exe found, adk setup still running"}
}
```

We prepared an "offline" ADK setup and put all files in the ADK Iso.

In addition, the following windows features are installed to allow the successfull installation of the ADK & SCCM Setup :
- Remote Differential Compression
- WSUS (updateServices-api)
- BITS with all subcomponents
- WMI compatibility in IIS

Also the necessary firewall rules are added for SQL. Although SQL is installed locally, the monitoring pane in SCCM will complain if these rules are not set.

The ADK Setup is kicked off and we monitor the existance of the "adksetup.exe" process before continuing the rest of the script.

## Step 5 ##

```posh
Set-VMDvdDrive -VMname $vmname -Path D:\Sources\CMUpdates.iso

$script = {
  param($remotevmname)
  new-item -Path C:\CMUpdates -itemtype Directory
  new-item -Path C:\WSUS -itemtype Directory
  Copy-Item -Path e:\*.* -Destination C:\CMUpdates
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname


Set-VMDvdDrive -VMname $vmname -Path D:\Sources\mu_system_center_configuration_manager_endpoint_protection_version_1606_x86_x64_dvd_9678361.iso

$script = {
  param($remotevmname)
  e:\SMSSETUP\Bin\X64\setup /script c:\CMUpdates\ConfigMgrAutoSave.ini 
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
    if ((Get-Process | ? {$_.Name -like "setupwpf" }).count -eq 0)
    {
        write-host "setupwpf.exe not found, assuming sccm setup finished"
    }
    else 
    {Write-Error "setupwpf.exe found, sccm setup still running"}

}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

The CMUpdates.iso contains the additional setup files that SCCM downloads during a fresh install. We just added them all to our own iso so we can reference them during setup.
The content is copied over to a local file.
Then we can finally mount the SCCM ISO and start setup with the use of the ini file.
Similar to an unattended SQL Setup, this INI file is created at the end when running the SCCM Installation wizard.

we monitor the setupwpf process for completion of the SCCM Installation.

## Step 6 ##

```posh
$script = {
  param($remotevmname)
  Install-WindowsFeature -ComputerName $remotevmname -ConfigurationFilePath C:\CMUpdates\DeploymentConfigTemplate.xml
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

Allthough we already installed the WSUS components a few steps earlier, you still need to finish the wsus wizard before we can actually use it in a Software update point.
this is done with the use of an XML file. The process is again similar to SQL and SCCM. Run the setup once manually and at the end you can export an XML file to automate the setup. Once you have the XML, add it to one of your ISO's so you can re-use it. I choose to add it to my CMUpdates ISO but any other will do just fine.

## Step 7 ##

```posh
$script = {
  param($remotevmname)
  Start-Process "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\Microsoft.ConfigurationManagement.exe"
  start-sleep -Seconds 30
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  #Step 1 - Import the Configuration Manager module                                                                
  Import-Module $env:SMS_ADMIN_UI_PATH.Replace("\bin\i386","\bin\configurationmanager.psd1") -force                       
  $SiteCode = Get-PSDrive -PSProvider CMSITE                                                                        
  Set-Location "$($SiteCode.Name):\Application"                                                                                
  new-Item Microsoft                                                                                                
  new-Item Adobe                                                                                                    
  new-Item IBM                                                                                                      
  new-Item Other                                                                                                    
  new-Item In-House                                                                                                 
  Set-Location "$($SiteCode.Name):\Package"                                                                                
  new-item Microsoft                                                                                                
  new-item Adobe                                                                                                    
  New-item IBM                                                                                                      
  New-Item Other                                                                                                    
  New-Item In-House                                                                                                 
  Set-Location "$($SiteCode.Name):\OperatingSystemImage"                                                                                
  new-item 'Windows 7'                                                                                              
  new-item 'Windows 8.1'                                                                                            
  new-item 'Windows 10'                                                                                             
  new-item 'Windows Server 2012 R2'                                                                                 
  new-item 'Windows Server 2016'                                                                                    
  Set-Location "$($SiteCode.Name):\OperatingSystemInstaller"                                                                         
  new-item 'Windows 7'                                                                                              
  new-item 'Windows 8.1'                                                                                            
  new-item 'Windows 10'                                                                                             
  new-item 'Windows Server 2012 R2'                                                                                 
  new-item 'Windows Server 2016'                                                                                     

  Set-Location "$($SiteCode.Name):\DeviceCollection"
  New-item '_Builtin'
  New-item 'Software Updates'
  New-item 'Operating Systems'
  New-item 'Application Management'
  New-item 'Compliance Settings'
  New-item 'Client Settings'
  New-item 'Maintenance Windows'
  New-item 'Operational'

  Set-Location "$($SiteCode.Name):\UserCollection"
  New-item '_Builtin'
  New-item 'Application Management'
  New-item 'Compliance Settings'
  New-item 'Client Settings'


  #Create Default limiting collections
  $LimitingCollection = "All Systems"
  $Schedule = New-CMSchedule –RecurInterval Days –RecurCount 7

  $arrcollection =
  @(
    @{Name = "Clients | All"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Client = 1";Comment = "All devices detected by SCCM"; Foldername = "Operational"; }
    ,@{Name = "System Health | Clients Inactive"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_CH_ClientSummary on SMS_G_System_CH_ClientSummary.ResourceId = SMS_R_System.ResourceId where SMS_G_System_CH_ClientSummary.ClientActiveStatus = 0 and SMS_R_System.Client = 1 and SMS_R_System.Obsolete = 0";Comment = "All devices with SCCM client state inactive"; Foldername = "Operational"}
    ,@{Name = "System Health | Clients Active"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_CH_ClientSummary on SMS_G_System_CH_ClientSummary.ResourceId = SMS_R_System.ResourceId where SMS_G_System_CH_ClientSummary.ClientActiveStatus = 1 and SMS_R_System.Client = 1 and SMS_R_System.Obsolete = 0";Comment = "All devices with SCCM client state active"; Foldername = "Operational"}
    ,@{Name = "Clients version | Not Latest (1606)"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion != '5.00.8412.1006'";Comment = "All devices without SCCM client version 1511"; Foldername = "Operational"}
    ,@{Name = "Hardware Inventory | Clients Not Reporting since 14 Days"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where ResourceId in (select SMS_R_System.ResourceID from SMS_R_System inner join SMS_G_System_WORKSTATION_STATUS on SMS_G_System_WORKSTATION_STATUS.ResourceID = SMS_R_System.ResourceId where DATEDIFF(dd,SMS_G_System_WORKSTATION_STATUS.LastHardwareScan,GetDate()) > 14)";Comment = "All devices with SCCM client that have not communicated with hardware inventory over 14 days"; Foldername = "Operational"}
    ,@{Name = "Clients | No"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Client = 0 OR SMS_R_System.Client is NULL";Comment = "All devices without SCCM client installed"; Foldername = "Operational"}
    ,@{Name = "System Health | Obsolete"; Query = "select *  from  SMS_R_System where SMS_R_System.Obsolete = 1";Comment = "All devices with SCCM client state obsolete"; Foldername = "Operational"}
    ,@{Name = "Workstations | Active"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_CH_ClientSummary on SMS_G_System_CH_ClientSummary.ResourceId = SMS_R_System.ResourceId where (SMS_R_System.OperatingSystemNameandVersion like 'Microsoft Windows NT%Workstation%' or SMS_R_System.OperatingSystemNameandVersion = 'Windows 7 Entreprise 6.1') and SMS_G_System_CH_ClientSummary.ClientActiveStatus = 1 and SMS_R_System.Client = 1 and SMS_R_System.Obsolete = 0";Comment = "All workstations with active state"; Foldername = "Operational"}
    ,@{Name = "Laptops | All"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from  SMS_R_System inner join SMS_G_System_SYSTEM_ENCLOSURE on SMS_G_System_SYSTEM_ENCLOSURE.ResourceID = SMS_R_System.ResourceId where SMS_G_System_SYSTEM_ENCLOSURE.ChassisTypes in ('8', '9', '10', '11', '12', '14', '18', '21')";Comment = "All laptops"; Foldername = "Operational"}
    ,@{Name = "Servers | All"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server%'";Comment = "All servers"; Foldername = "Operational"}
    ,@{Name = "Servers | Physical"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ResourceId not in (select SMS_R_SYSTEM.ResourceID from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_R_System.IsVirtualMachine = 'True') and SMS_R_System.OperatingSystemNameandVersion like 'Microsoft Windows NT%Server%'";Comment = "All physical servers"; Foldername = "Operational"}
    ,@{Name = "Servers | Virtual"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.IsVirtualMachine = 'True' and SMS_R_System.OperatingSystemNameandVersion like 'Microsoft Windows NT%Server%'";Comment = "All virtual servers"; Foldername = "Operational"}
    ,@{Name = "Workstations | All"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation%'";Comment = "All workstations"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 7"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation 6.1%'";Comment = "All workstations with Windows 7 operating system"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 8"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation 6.2%'";Comment = "All workstations with Windows 8 operating system"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 8.1"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation 6.3%'";Comment = "All workstations with Windows 8.1 operating system"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows XP"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System   where OperatingSystemNameandVersion like '%Workstation 5.1%' or OperatingSystemNameandVersion like '%Workstation 5.2%'";Comment = "All workstations with Windows XP operating system"; Foldername = "Operational"}
    ,@{Name = "Servers | Windows 2008 and 2008 R2"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server 6.0%' or OperatingSystemNameandVersion like '%Server 6.1%'";Comment = "All servers with Windows 2008 or 2008 R2 operating system"; Foldername = "Operational"}
    ,@{Name = "Servers | Windows 2012 and 2012 R2"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server 6.2%' or OperatingSystemNameandVersion like '%Server 6.3%'";Comment = "All servers with Windows 2012 or 2012 R2 operating system"; Foldername = "Operational"}
    ,@{Name = "Servers | Windows 2003 and 2003 R2"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server 5.2%'";Comment = "All servers with Windows 2003 or 2003 R2 operating system"; Foldername = "Operational"}
    ,@{Name = "Systems | Created Since 24h"; Query = "select SMS_R_System.Name, SMS_R_System.CreationDate FROM SMS_R_System WHERE DateDiff(dd,SMS_R_System.CreationDate, GetDate()) <= 1";Comment = "All systems created in the last 24 hours"; Foldername = "Operational"}
    ,@{Name = "SCCM | Console"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_ADD_REMOVE_PROGRAMS on SMS_G_System_ADD_REMOVE_PROGRAMS.ResourceID = SMS_R_System.ResourceId where SMS_G_System_ADD_REMOVE_PROGRAMS.DisplayName like '%Configuration Manager Console%'";Comment = "All systems with SCCM console installed"; Foldername = "Operational"}
    ,@{Name = "SCCM | Site System"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from  SMS_R_System where SMS_R_System.SystemRoles = 'SMS Site System'";Comment = "All systems that is SCCM site system"; Foldername = "Operational"}
    ,@{Name = "SCCM | Site Servers"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from  SMS_R_System where SMS_R_System.SystemRoles = 'SMS Site Server'";Comment = "All systems that is SCCM site server"; Foldername = "Operational"}
    ,@{Name = "SCCM | Distribution Points"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from  SMS_R_System where SMS_R_System.SystemRoles = 'SMS Distribution Point'";Comment = "All systems that is SCCM distribution point"; Foldername = "Operational"}
    ,@{Name = "Windows Update Agent | Outdated Version Win7 RTM and Lower"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_WINDOWSUPDATEAGENTVERSION on SMS_G_System_WINDOWSUPDATEAGENTVERSION.ResourceID = SMS_R_System.ResourceId inner join SMS_G_System_OPERATING_SYSTEM on SMS_G_System_OPERATING_SYSTEM.ResourceID = SMS_R_System.ResourceId where SMS_G_System_WINDOWSUPDATEAGENTVERSION.Version < '7.6.7600.256' and SMS_G_System_OPERATING_SYSTEM.Version <= '6.1.7600'";Comment = "All systems with windows update agent with outdated version Win7 RTM and lower"; Foldername = "Operational"}
    ,@{Name = "Windows Update Agent | Outdated Version Win7 SP1 and Higher"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_WINDOWSUPDATEAGENTVERSION on SMS_G_System_WINDOWSUPDATEAGENTVERSION.ResourceID = SMS_R_System.ResourceId inner join SMS_G_System_OPERATING_SYSTEM on SMS_G_System_OPERATING_SYSTEM.ResourceID = SMS_R_System.ResourceId where SMS_G_System_WINDOWSUPDATEAGENTVERSION.Version < '7.6.7600.320' and SMS_G_System_OPERATING_SYSTEM.Version >= '6.1.7601'";Comment = "All systems with windows update agent with outdated version Win7 SP1 and higher"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Android"; Query = "SELECT SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client FROM SMS_R_System INNER JOIN SMS_G_System_DEVICE_OSINFORMATION ON SMS_G_System_DEVICE_OSINFORMATION.ResourceID = SMS_R_System.ResourceId WHERE SMS_G_System_DEVICE_OSINFORMATION.Platform like 'Android%'";Comment = "All Android modible devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Iphone"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_DEVICE_COMPUTERSYSTEM on SMS_G_System_DEVICE_COMPUTERSYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_DEVICE_COMPUTERSYSTEM.DeviceModel like '%Iphone%'";Comment = "All Iphone modible devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Ipad"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_DEVICE_COMPUTERSYSTEM on SMS_G_System_DEVICE_COMPUTERSYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_DEVICE_COMPUTERSYSTEM.DeviceModel like '%Ipad%'";Comment = "All Ipad mobile devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Windows Phone 8"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_DEVICE_OSINFORMATION on SMS_G_System_DEVICE_OSINFORMATION.ResourceID = SMS_R_System.ResourceId where SMS_G_System_DEVICE_OSINFORMATION.Platform = 'Windows Phone' and SMS_G_System_DEVICE_OSINFORMATION.Version like '8.0%'";Comment = "All Windows 8 mobile devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Windows Phone 8.1"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_DEVICE_OSINFORMATION on SMS_G_System_DEVICE_OSINFORMATION.ResourceID = SMS_R_System.ResourceId where SMS_G_System_DEVICE_OSINFORMATION.Platform = 'Windows Phone' and SMS_G_System_DEVICE_OSINFORMATION.Version like '8.1%'";Comment = "All Windows 8.1 mobile devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Microsoft Surface"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Model like '%Surface%'";Comment = "All Windows RT mobile devices"; Foldername = "Operational"}
    ,@{Name = "System Health | Disabled"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.UserAccountControl ='4098'";Comment = "All systems with client state disabled"; Foldername = "Operational"}
    ,@{Name = "Systems | x86"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceID = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.SystemType = 'X86-based PC'";Comment = "All systems with 32-bit system type"; Foldername = "Operational"}
    ,@{Name = "Systems | x64"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceID = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.SystemType = 'X64-based PC'";Comment = "All systems with 64-bit system type"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU1"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1203'";Comment = "All systems with SCCM client version R2 CU1 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU2"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1303'";Comment = "All systems with SCCM client version R2 CU2 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU3"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion like '5.00.7958.14%'";Comment = "All systems with SCCM client version R2 CU3 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU4"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1501'";Comment = "All systems with SCCM client version R2 CU4 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU5"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1604'";Comment = "All systems with SCCM client version R2 CU5 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU0"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1000'";Comment = "All systems with SCCM client version R2 CU0 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 SP1"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8239.1000'";Comment = "All systems with SCCM client version R2 SP1 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 SP1 CU1"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8239.1203'";Comment = "All systems with SCCM client version R2 SP1 CU1 installed"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation 10.0%'";Comment = "All workstations with Windows 10 operating system"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 SP1 CU2"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8239.1301'";Comment = "All systems with SCCM client version R2 SP1 CU2 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | 1511"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8325.1000'";Comment = "All systems with SCCM client version 1511 installed"; Foldername = "Operational"}
    ,@{Name = "Laptops | Dell"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Manufacturer like '%Dell%'";Comment = "All laptops with Dell manufacturer"; Foldername = "Operational"}
    ,@{Name = "Laptops | Lenovo"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Manufacturer like '%Lenovo%'";Comment = "All laptops with Lenovo manufacturer"; Foldername = "Operational"}
    ,@{Name = "Laptops | HP"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Manufacturer like '%HP%' or SMS_G_System_COMPUTER_SYSTEM.Manufacturer like '%Hewlett-Packard%'";Comment = "All laptops with HP manufacturer"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 SP1 CU3"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8239.1403'";Comment = "All systems with SCCM client version R2 SP1 CU3 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | 1602"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8355.1000'";Comment = "All systems with SCCM client version 1602 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | 1606"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion like '5.00.8412.100%'";Comment = "All systems with SCCM client version 1606 installed"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Microsoft Surface 3"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Model = 'Surface Pro 3' OR SMS_G_System_COMPUTER_SYSTEM.Model = 'Surface 3'";Comment = "All Microsoft Surface 3"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Microsoft Surface 4"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Model = 'Surface Pro 4'";Comment = "All Microsoft Surface 4"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Windows Phone 10"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_DEVICE_OSINFORMATION on SMS_G_System_DEVICE_OSINFORMATION.ResourceID = SMS_R_System.ResourceId where SMS_G_System_DEVICE_OSINFORMATION.Platform = 'Windows Phone' and SMS_G_System_DEVICE_OSINFORMATION.Version like '10%'";Comment = "All Windows Phone 10"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | All"; Query = "select * from SMS_R_System where SMS_R_System.ClientType = 3";Comment = "All Mobile Devices"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 v1507"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Build like '10.0.10240%'";Comment = "All workstations with Windows 10 operating system v1507"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 v1511"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Build like '10.0.10586%'";Comment = "All workstations with Windows 10 operating system v1511"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 v1607"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Build like '10.0.14393%'";Comment = "All workstations with Windows 10 operating system v1607"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Current Branch (CB)"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.OperatingSystemNameandVersion like '%Workstation 10.0%' and SMS_R_System.OSBranch = '0'";Comment = "All workstations with Windows 10 CB"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Current Branch for Business (CBB)"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.OperatingSystemNameandVersion like '%Workstation 10.0%' and SMS_R_System.OSBranch = '1'";Comment = "All workstations with Windows 10 CBB"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Long Term Servicing Branch (LTSB)"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.OperatingSystemNameandVersion like '%Workstation 10.0%' and SMS_R_System.OSBranch = '2'";Comment = "All workstations with Windows 10 LTSB"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Support State - Current"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System  LEFT OUTER JOIN SMS_WindowsServicingStates ON SMS_WindowsServicingStates.Build = SMS_R_System.build01 AND SMS_WindowsServicingStates.branch = SMS_R_System.osbranch01 where SMS_WindowsServicingStates.State = '2'";Comment = "Windows 10 Support State - Current"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Support State - Expired Soon"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System  LEFT OUTER JOIN SMS_WindowsServicingStates ON SMS_WindowsServicingStates.Build = SMS_R_System.build01 AND SMS_WindowsServicingStates.branch = SMS_R_System.osbranch01 where SMS_WindowsServicingStates.State = '3'";Comment = "Windows 10 Support State - Expired Soon"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Support State - Expired"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System  LEFT OUTER JOIN SMS_WindowsServicingStates ON SMS_WindowsServicingStates.Build = SMS_R_System.build01 AND SMS_WindowsServicingStates.branch = SMS_R_System.osbranch01 where SMS_WindowsServicingStates.State = '4'";Comment = "Windows 10 Support State - Expired"; Foldername = "Operational"}
    ,@{Name = "Servers | Windows 2016"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server 10%'";Comment = "All Servers with Windows 2016"; Foldername = "Operational"}
    ,@{Name = "Clients Version | 1610"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion like '5.00.8458.100%'";Comment = "All systems with SCCM client version 1610 installed"; Foldername = "Operational"}
    ,@{Name = "Windows 10 1511 OSD"; Query = ""; Comment = "Windows 10 1511 OSD" ;Foldername = "Operating Systems"}
  )

  #Create Collection
  foreach ($Collection in $Arrcollection)
    {
    New-CMDeviceCollection -Name $Collection.Name -Comment $Collection.Comment -LimitingCollectionName $LimitingCollection -RefreshSchedule $Schedule -RefreshType 2 | Out-Null
    try
        {
        Add-CMDeviceCollectionQueryMembershipRule -CollectionName $Collection.Name -QueryExpression $Collection.Query -RuleName $Collection.Name
        Write-host *** Collection $Collection.Name created ***
        }
    catch
        {}
    #Move the collection to the right folder
    $FolderPath = "CBL:\DeviceCollection\" + $Collection.FolderName
    Move-CMObject -FolderPath $FolderPath -InputObject (Get-CMDeviceCollection -Name $Collection.Name)
    }
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

Allright ! Now that our SCCM Server is up and running we can continue configuring it.
Luckily Microsoft provided a lot of SCCM Powershell commandlets that we can leverage. However, during my initial tests none of these commands seemed to work :(
After some testing I noticed that launching the console seemed to "activate" the cmdlets so that's what happens in the first script block. There could very well be more elegant ways to fix this but lack of time made me go this way.

After importing the SCCM Powershell module and connecting to the SCCM Psdrive we continue with creating a set of subfolders in the console. After doing many implementations we found this layout a good starting point. For the Packages and Applications we create a subfolder for the "major" software vendors and in house developped software. 

For OSD we add all the currently supported OS's and for user and device collections we create some placeholder folders for the major types of deployments.

The next step is taken and modified from [Benoit Lecours's blog](https://www.systemcenterdudes.com/powershell-script-create-set-maintenance-collections/). He has written a script that creates a set of operational collections that are very usefull in every environment !! However I tried to optimize the script a bit by creating, adding the query and moving them to the correct folder all in one go where Benoit's current script has separate steps for this. The original script can be found on the [technet gallery](https://gallery.technet.microsoft.com/Set-of-Operational-SCCM-19fa8178).

My version of his script puts all collections in an array with the following properties :
- Name
- Query
- Comments
- Foldername

Adding a row with these properties results in your collection being added to the folder of your choice.

## Step 8 ##

```posh
$script = {

  Install-WindowsFeature FS-DFS-Namespace, FS-DFS-Replication, RSAT-DFS-Mgmt-Con

  New-Item -ItemType 'directory' -Path 'D:\dfsroot'
   
  Set-Location D:\dfsroot

  New-Item -ItemType 'directory' -Path 'Application Management'
  New-Item -ItemType 'directory' -Path 'Compliance Settings'
  New-Item -ItemType 'directory' -Path 'Export'
  New-Item -ItemType 'directory' -Path 'Import'
  New-Item -ItemType 'directory' -Path 'Logs'
  New-Item -ItemType 'directory' -Path 'Logs\Archived Logs'
  New-Item -ItemType 'directory' -Path 'Logs\Task Sequence Error Logs'
  New-Item -ItemType 'directory' -Path 'Operating Systems'
  New-Item -ItemType 'directory' -Path 'Resources'
  New-Item -ItemType 'directory' -Path 'Resources\tools'
  New-Item -ItemType 'directory' -Path 'Resources\scripts'
  New-Item -ItemType 'directory' -Path 'Resources\hotfixes'
  New-Item -ItemType 'directory' -Path 'Resources\CMCurrent-Branch Updates'
  New-Item -ItemType 'directory' -Path 'Software Updates'


  New-Item -ItemType 'directory' -Path 'Application Management\Packages\Adobe'
  New-Item -ItemType 'directory' -Path 'Application Management\Packages\IBM'
  New-Item -ItemType 'directory' -Path 'Application Management\Packages\In-House'
  New-Item -ItemType 'directory' -Path 'Application Management\Packages\Microsoft' 
  New-Item -ItemType 'directory' -Path 'Application Management\Packages\Other'

  New-Item -ItemType 'directory' -Path 'Application Management\Applications\Adobe'
  New-Item -ItemType 'directory' -Path 'Application Management\Applications\IBM'
  New-Item -ItemType 'directory' -Path 'Application Management\Applications\In-House'
  New-Item -ItemType 'directory' -Path 'Application Management\Applications\Microsoft' 
  New-Item -ItemType 'directory' -Path 'Application Management\Applications\Other'

  New-Item -ItemType 'directory' -Path 'Export\Task Sequences Monitor'
  New-Item -ItemType 'directory' -Path 'Import\Applications'
  New-Item -ItemType 'directory' -Path 'Import\Baselines'
  New-Item -ItemType 'directory' -Path 'Import\Hardware Inventory MOFs'
  New-Item -ItemType 'directory' -Path 'Import\Report Definitions'
  New-Item -ItemType 'directory' -Path 'Import\Tasksequences'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Boot Images'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Branding'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Captures'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Task Sequence Media'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Virtual Hard Disks'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Scripts'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Scripts\Supported Models'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Scripts\Supported Models\SetDriverCategory'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Scripts\Supported Models\SetHWApps'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages\Dell'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages\HP'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages\Lenovo'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages\Microsoft'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers\Dell'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers\HP'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers\Lenovo'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers\Microsoft'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows 7'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows 8.1'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows 10'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows Windows Server 2012 R2'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows Server 2016'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows 7'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows 8.1'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows 10'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows Windows Server 2012 R2'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows Server 2016'

  New-Item -ItemType 'directory' -Path 'Software Updates\Deployment Packages'
  New-Item -ItemType 'directory' -Path 'Software Updates\SCUP'
  New-Item -ItemType 'directory' -Path 'Software Updates\ExportSynchronizedProducts'

}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

The end is in sight! This block creates, just like the previous one, a set of folders on a share of your choice. Preferrably even a DFS share as that will be most future proof. We tried to mimic the layout of the console as much as possible in our "sources" share so items are easy to locate.

## Step 9 ##

```posh
$script = {
  Import-Module $env:SMS_ADMIN_UI_PATH.Replace("\bin\i386","\bin\configurationmanager.psd1") -force                       
  $SiteCode = Get-PSDrive -PSProvider CMSITE                                                                        

  Set-Location ($SiteCode.Name +":")
  #$pwd = read-host -AsSecureString

  #create new credential variable for CMInstall
  $NAA = "Training\SCCM_NAA"
  $pwd = ConvertTo-SecureString "T0psecret" -AsPlainText -Force

  New-CMAccount -Name $NAA -Password $pwd -Sitecode $SiteCode.Name

  #make the created user account your new Network Access Account

  $component = gwmi -class SMS_SCI_ClientComp -Namespace "root\sms\site_$($SiteCode.name)"  | Where-Object {$_.ItemName -eq "Software Distribution"}
  $component
  $props = $component.PropLists

  $prop = $props | where {$_.PropertyListName -eq "Network Access User Names"}

  $new = [WmiClass] "root\sms\site_$($SiteCode.name):SMS_EmbeddedPropertyList"
  $embeddedproperylist = $new.CreateInstance()

  $embeddedproperylist.PropertyListName = "Network Access User Names"
  $embeddedproperylist.Values = $NAA
  $component.PropLists += $embeddedproperylist
  $component.Put() | Out-Null

  # Configure boundary + boundarygroup

  $BGName = "Labo"
  New-CMBoundary -type IPRange -Value "172.30.150.1-172.30.150.254" -name "Trainig IP range"
  New-CMBoundaryGroup -Name $BGName
  Add-CMBoundaryToGroup -Boundaryname "Trainig IP range" -BoundaryGroupName $BGName
  Set-CMDistributionPoint -SiteSystemServerName "Primary.Training.Local" -AddBoundaryGroupName $BGName

}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  $lastsccmbuild = get-wmiobject -namespace root\sms\site_cbl -class SMS_CM_Updatepackages | Select-Object fullversion,state | Sort-Object fullversion -Descending
  if ($lastsccmbuild[0].state -eq "262146")
    {write-host "Last build available... Starting the update"
    $filter = "Fullversion='" + $lastsccmbuild[0].fullversion +"'"
    $Update = get-wmiobject -namespace root\sms\site_cbl -class SMS_CM_Updatepackages -filter $filter  
    $Update.UpdatePrereqAndStateFlags(0,2)
    }
  else
    {write-error "Last $lastsccmbuild not available yet"
    }
}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

We finalize the SCCM Configuratin with the configuration of the Network access account and boundaries. I documented this step already in a previous [blog-post](http://www.oscc.be/sccm/configmgr/powershell/naa/Set-NAA-using-Powershell-in-CB/).

Last, but definitely not leastis a rather interesting block. It can be skipped as this one takes time.
The moment SCCM is installed it will start downloading the bits for the newer builds. At the time of writing this was the 1610 release. Obviously this takes a while to complete.

This last block is checking in WMI, in the "SMS_CM_UpdatePackages" for the state of the most recent full version available. Once that state reaches "262146" we know it has downloaded successfully and is ready to install.
Once completed, the installation is kicked off with the WMI method "UpdatePrereqAndStateFlags" and the script finishes.

If you want to test this, be patient as downloading takes some time !

## Wrap up ##

You can find the full script here at the end of the blogpost.

Part 4 will tie the previous posts together and explain how we setup the Hyper-V server and launch these scripts..

As usual, feel free to comment or let me know if something isn't clear for you or isn't working out at all.

```posh
$VMName = "SCCM_Primary"
$Memory = 12GB
$NewVHD = "D:\VM\VHD\" +$VMName + ".vhdx"
$VHDSize = "100000000000"
$Switchexternal = "externalSwitch"
$Switchprivate = "privateSwitch"
$Cpu=4
$IP = "172.30.150.2"
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

NEW-VM -Name $Vmname -MemoryStartupBytes $Memory -VHDPath $NewVHD -Generation 2 -Switchname $Switchprivate
SET-VMProcessor -VMName $VMName –count $Cpu 
Set-VM -VMName $VMName -MemoryMaximumBytes $memory

start-vm $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

Stop-VM –Name $vmname
start-vm $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$password = ConvertTo-SecureString $Localadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("administrator", $password )

$NewVHD = "D:\VM\VHD\" +$VMName + "_D.vhdx"
$VHDSize = "100000000000"

New-VHD -Path $newvhd -SizeBytes $vhdsize
Add-VMHardDiskDrive -VMName $VMName -Path $newvhd

##Add extra harddisk

$script = {
  param($remotevmname)
  Get-Disk | `
  Where-Object partitionstyle -eq 'raw' | `
  Initialize-Disk -PartitionStyle MBR -PassThru | `
  New-Partition -AssignDriveLetter -UseMaximumSize | `
  Format-Volume -FileSystem NTFS -NewFileSystemLabel "Data" -Confirm:$false
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  rename-computer -newname $remotevmname
  restart-computer
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

Add-VMDvdDrive -vmname $VMName

$script = {
  param($remotevmname)
  $first = get-netadapter
  Rename-NetAdapter -name $first.Name -NewName "Private"
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname,$myip)
  New-NetIPAddress -InterfaceAlias "Private" -IPAddress $myip -DefaultGateway 172.30.150.254 -PrefixLength 24 
  Set-DnsClientServerAddress -InterfaceAlias "Private" -ServerAddresses 172.30.150.1
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$IP

$script = {
  if ((Get-DnsClientServerAddress -InterfaceAlias "Private" | ? {$_.AddressFamily -eq '2'} ).ServerAddresses[0] -ne "172.30.150.1")
  {
    Write-Error "First dns server not set correctly"
  }
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname,$domainpwd)
  $domain = "Training.Local"
  $password = $domainpwd | ConvertTo-SecureString -asPlainText -Force
  $username = "$domain\administrator" 
  $credential = New-Object System.Management.Automation.PSCredential($username,$password)
  Add-Computer -DomainName $domain -Credential $credential
  restart-computer
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname, $Domainadminpwd

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$password = ConvertTo-SecureString $Domainadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("training\administrator", $password )

Set-VMDvdDrive -VMname $vmname -Path D:\Sources\en_windows_server_2016_x64_dvd_9327751.iso

$script = {
  param($remotevmname)
  Install-WindowsFeature Net-Framework-Core -source e:\sources\sxs
  net localgroup administrators /add Training\CMinstall
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  Add-WindowsFeature RSAT-ADDS, RSAT-ADDS-Tools
  Import-Module ActiveDirectory
  $computer = $remotevmname + "$"
  ADD-ADGroupMember "CmServers" –members $computer
  restart-computer
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

$password = ConvertTo-SecureString $Cminstallpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("training\CMInstall", $password )

Set-VMDvdDrive -VMname $vmname -Path D:\Sources\en_sql_server_2016_standard_with_service_pack_1_x64_dvd_9540929.iso

$script = {
    if (test-path "e:\setup.exe")
    { write-host "found setup.exe, launching sql setup"}
    else
    {
        write-error "e:\setup.exe not found"
    }
}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  new-item -Path C:\SQLUpdates -itemtype Directory
  e:\setup.exe /ACTION=Install /IACCEPTSQLSERVERLICENSETERMS /UpdateEnabled=1 /UpdateSource=c:\SQLUpdates /features="SQLEngine" "RS" "TOOLS" /installshareddir="c:\SQL\(X64)" /InstallsharedwowDir="c:\SQL\(X86)" /instancedir="c:\SQL\(X64)" /Instancename=MSSQLSERVER /q /AGTSVCACCOUNT="NT Authority\System" /INSTALLSQLDATADIR="c:\SQL\(X64)" /SQLBACKUPDIR="c:\MSSQLSERVER\BACKUP" /SQLCOLLATION=SQL_Latin1_General_CP1_CI_AS /SQLSYSADMINACCOUNTS="Training\domain admins" "Primary\administrator" "Training\cminstall" /SQLSVCACCOUNT="NT Authority\System" /SQLTEMPDBDIR="c:\MSSQLSERVER\TEMPDB" /SQLTEMPDBLOGDIR="c:\MSSQLSERVER\LOGS" /SQLUSERDBDIR="c:\MSSQLSERVER\USERDB" /SQLUSERDBLOGDIR="c:\MSSQLSERVER\Logs"  /RSSVCACCOUNT="NT AUTHORITY\SYSTEM" /INDICATEPROGRESS 
  start-sleep -seconds 30
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
    if ((Get-Process | ? {$_.Name -like "*setup*" }).count -eq 0)
    {
        write-host "setup.exe not found, assuming sql setup finished"
    }
    else 
    {Write-Error "setup.exe found, sql setup still running"}


}
loopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Set-VMDvdDrive -VMname $vmname -Path D:\Sources\SSMS.iso

$script = {
  param($remotevmname)
  e:\SSMS-Setup-ENU /Install /quiet /norestart
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
    start-sleep -Seconds 3
    if ((Get-Process | ? {$_.Name -eq "SSMS-Setup-ENU"}).count -eq 0)
    {
        write-host "setup.exe not found, assuming sql setup finished"
    }
    else 
    {Write-Error "setup.exe found, sql setup still running"}


}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Set-VMDvdDrive -VMname $vmname -Path D:\Sources\ADK1607.iso

$script = {
  param($remotevmname)
  add-windowsfeature rdc
  add-windowsfeature UpdateServices-API
  Add-windowsfeature BITS, BITS-IIS-EXT, RSAT-BITS-SERVER
  Add-windowsfeature WEB-MGMT-Compat, WEB-WMI
  New-NetFirewallRule -DisplayName "SQL Server" -Direction Inbound –Protocol TCP –LocalPort 1433 -Action allow
  New-NetFirewallRule -DisplayName "SQL Admin Connection" -Direction Inbound –Protocol TCP –LocalPort 1434 -Action allow
  New-NetFirewallRule -DisplayName "SQL Database Management" -Direction Inbound –Protocol UDP –LocalPort 1434 -Action allow
  New-NetFirewallRule -DisplayName "SQL Service Broker" -Direction Inbound –Protocol TCP –LocalPort 4022 -Action allow
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  e:\adksetup.exe /log c:\cmupdates\adksetup.log /Features OptionId.DeploymentTools OptionId.UserStateMigrationTool OptionId.WindowsPreinstallationEnvironment /norestart /ceip off /quiet
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
    if ((Get-Process | ? {$_.Name -eq "adksetup.exe" -or $_.Name -like "*msiexec*"}).count -eq 0)
    {
        write-host "adksetup.exe not found, assuming adk setup finished"
    }
    else 
    {Write-Error "adksetup.exe found, adk setup still running"}
}
loopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

Set-VMDvdDrive -VMname $vmname -Path D:\Sources\CMUpdates.iso

$script = {
  param($remotevmname)
  new-item -Path C:\CMUpdates -itemtype Directory
  new-item -Path C:\WSUS -itemtype Directory
  Copy-Item -Path e:\*.* -Destination C:\CMUpdates
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname


Set-VMDvdDrive -VMname $vmname -Path D:\Sources\mu_system_center_configuration_manager_endpoint_protection_version_1606_x86_x64_dvd_9678361.iso

$script = {
  param($remotevmname)
  e:\SMSSETUP\Bin\X64\setup /script c:\CMUpdates\ConfigMgrAutoSave.ini 
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
    if ((Get-Process | ? {$_.Name -like "setupwpf" }).count -eq 0)
    {
        write-host "setupwpf.exe not found, assuming sccm setup finished"
    }
    else 
    {Write-Error "setupwpf.exe found, sccm setup still running"}

}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  Install-WindowsFeature -ComputerName $remotevmname -ConfigurationFilePath C:\CMUpdates\DeploymentConfigTemplate.xml
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname


$script = {
  param($remotevmname)
  Start-Process "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\Microsoft.ConfigurationManagement.exe"
  start-sleep -Seconds 30
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  #Step 1 - Import the Configuration Manager module                                                                
  Import-Module $env:SMS_ADMIN_UI_PATH.Replace("\bin\i386","\bin\configurationmanager.psd1") -force                       
  $SiteCode = Get-PSDrive -PSProvider CMSITE                                                                        
  Set-Location "$($SiteCode.Name):\Application"                                                                                
  new-Item Microsoft                                                                                                
  new-Item Adobe                                                                                                    
  new-Item IBM                                                                                                      
  new-Item Other                                                                                                    
  new-Item In-House                                                                                                 
  Set-Location "$($SiteCode.Name):\Package"                                                                                
  new-item Microsoft                                                                                                
  new-item Adobe                                                                                                    
  New-item IBM                                                                                                      
  New-Item Other                                                                                                    
  New-Item In-House                                                                                                 
  Set-Location "$($SiteCode.Name):\OperatingSystemImage"                                                                                
  new-item 'Windows 7'                                                                                              
  new-item 'Windows 8.1'                                                                                            
  new-item 'Windows 10'                                                                                             
  new-item 'Windows Server 2012 R2'                                                                                 
  new-item 'Windows Server 2016'                                                                                    
  Set-Location "$($SiteCode.Name):\OperatingSystemInstaller"                                                                         
  new-item 'Windows 7'                                                                                              
  new-item 'Windows 8.1'                                                                                            
  new-item 'Windows 10'                                                                                             
  new-item 'Windows Server 2012 R2'                                                                                 
  new-item 'Windows Server 2016'                                                                                     

  Set-Location "$($SiteCode.Name):\DeviceCollection"
  New-item '_Builtin'
  New-item 'Software Updates'
  New-item 'Operating Systems'
  New-item 'Application Management'
  New-item 'Compliance Settings'
  New-item 'Client Settings'
  New-item 'Maintenance Windows'
  New-item 'Operational'

  Set-Location "$($SiteCode.Name):\UserCollection"
  New-item '_Builtin'
  New-item 'Application Management'
  New-item 'Compliance Settings'
  New-item 'Client Settings'


  #Create Default limiting collections
  $LimitingCollection = "All Systems"
  $Schedule = New-CMSchedule –RecurInterval Days –RecurCount 7

  $arrcollection =
  @(
    @{Name = "Clients | All"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Client = 1";Comment = "All devices detected by SCCM"; Foldername = "Operational"; }
    ,@{Name = "System Health | Clients Inactive"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_CH_ClientSummary on SMS_G_System_CH_ClientSummary.ResourceId = SMS_R_System.ResourceId where SMS_G_System_CH_ClientSummary.ClientActiveStatus = 0 and SMS_R_System.Client = 1 and SMS_R_System.Obsolete = 0";Comment = "All devices with SCCM client state inactive"; Foldername = "Operational"}
    ,@{Name = "System Health | Clients Active"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_CH_ClientSummary on SMS_G_System_CH_ClientSummary.ResourceId = SMS_R_System.ResourceId where SMS_G_System_CH_ClientSummary.ClientActiveStatus = 1 and SMS_R_System.Client = 1 and SMS_R_System.Obsolete = 0";Comment = "All devices with SCCM client state active"; Foldername = "Operational"}
    ,@{Name = "Clients version | Not Latest (1606)"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion != '5.00.8412.1006'";Comment = "All devices without SCCM client version 1511"; Foldername = "Operational"}
    ,@{Name = "Hardware Inventory | Clients Not Reporting since 14 Days"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where ResourceId in (select SMS_R_System.ResourceID from SMS_R_System inner join SMS_G_System_WORKSTATION_STATUS on SMS_G_System_WORKSTATION_STATUS.ResourceID = SMS_R_System.ResourceId where DATEDIFF(dd,SMS_G_System_WORKSTATION_STATUS.LastHardwareScan,GetDate()) > 14)";Comment = "All devices with SCCM client that have not communicated with hardware inventory over 14 days"; Foldername = "Operational"}
    ,@{Name = "Clients | No"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Client = 0 OR SMS_R_System.Client is NULL";Comment = "All devices without SCCM client installed"; Foldername = "Operational"}
    ,@{Name = "System Health | Obsolete"; Query = "select *  from  SMS_R_System where SMS_R_System.Obsolete = 1";Comment = "All devices with SCCM client state obsolete"; Foldername = "Operational"}
    ,@{Name = "Workstations | Active"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_CH_ClientSummary on SMS_G_System_CH_ClientSummary.ResourceId = SMS_R_System.ResourceId where (SMS_R_System.OperatingSystemNameandVersion like 'Microsoft Windows NT%Workstation%' or SMS_R_System.OperatingSystemNameandVersion = 'Windows 7 Entreprise 6.1') and SMS_G_System_CH_ClientSummary.ClientActiveStatus = 1 and SMS_R_System.Client = 1 and SMS_R_System.Obsolete = 0";Comment = "All workstations with active state"; Foldername = "Operational"}
    ,@{Name = "Laptops | All"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from  SMS_R_System inner join SMS_G_System_SYSTEM_ENCLOSURE on SMS_G_System_SYSTEM_ENCLOSURE.ResourceID = SMS_R_System.ResourceId where SMS_G_System_SYSTEM_ENCLOSURE.ChassisTypes in ('8', '9', '10', '11', '12', '14', '18', '21')";Comment = "All laptops"; Foldername = "Operational"}
    ,@{Name = "Servers | All"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server%'";Comment = "All servers"; Foldername = "Operational"}
    ,@{Name = "Servers | Physical"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ResourceId not in (select SMS_R_SYSTEM.ResourceID from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_R_System.IsVirtualMachine = 'True') and SMS_R_System.OperatingSystemNameandVersion like 'Microsoft Windows NT%Server%'";Comment = "All physical servers"; Foldername = "Operational"}
    ,@{Name = "Servers | Virtual"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.IsVirtualMachine = 'True' and SMS_R_System.OperatingSystemNameandVersion like 'Microsoft Windows NT%Server%'";Comment = "All virtual servers"; Foldername = "Operational"}
    ,@{Name = "Workstations | All"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation%'";Comment = "All workstations"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 7"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation 6.1%'";Comment = "All workstations with Windows 7 operating system"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 8"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation 6.2%'";Comment = "All workstations with Windows 8 operating system"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 8.1"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation 6.3%'";Comment = "All workstations with Windows 8.1 operating system"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows XP"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System   where OperatingSystemNameandVersion like '%Workstation 5.1%' or OperatingSystemNameandVersion like '%Workstation 5.2%'";Comment = "All workstations with Windows XP operating system"; Foldername = "Operational"}
    ,@{Name = "Servers | Windows 2008 and 2008 R2"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server 6.0%' or OperatingSystemNameandVersion like '%Server 6.1%'";Comment = "All servers with Windows 2008 or 2008 R2 operating system"; Foldername = "Operational"}
    ,@{Name = "Servers | Windows 2012 and 2012 R2"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server 6.2%' or OperatingSystemNameandVersion like '%Server 6.3%'";Comment = "All servers with Windows 2012 or 2012 R2 operating system"; Foldername = "Operational"}
    ,@{Name = "Servers | Windows 2003 and 2003 R2"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server 5.2%'";Comment = "All servers with Windows 2003 or 2003 R2 operating system"; Foldername = "Operational"}
    ,@{Name = "Systems | Created Since 24h"; Query = "select SMS_R_System.Name, SMS_R_System.CreationDate FROM SMS_R_System WHERE DateDiff(dd,SMS_R_System.CreationDate, GetDate()) <= 1";Comment = "All systems created in the last 24 hours"; Foldername = "Operational"}
    ,@{Name = "SCCM | Console"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_ADD_REMOVE_PROGRAMS on SMS_G_System_ADD_REMOVE_PROGRAMS.ResourceID = SMS_R_System.ResourceId where SMS_G_System_ADD_REMOVE_PROGRAMS.DisplayName like '%Configuration Manager Console%'";Comment = "All systems with SCCM console installed"; Foldername = "Operational"}
    ,@{Name = "SCCM | Site System"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from  SMS_R_System where SMS_R_System.SystemRoles = 'SMS Site System'";Comment = "All systems that is SCCM site system"; Foldername = "Operational"}
    ,@{Name = "SCCM | Site Servers"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from  SMS_R_System where SMS_R_System.SystemRoles = 'SMS Site Server'";Comment = "All systems that is SCCM site server"; Foldername = "Operational"}
    ,@{Name = "SCCM | Distribution Points"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from  SMS_R_System where SMS_R_System.SystemRoles = 'SMS Distribution Point'";Comment = "All systems that is SCCM distribution point"; Foldername = "Operational"}
    ,@{Name = "Windows Update Agent | Outdated Version Win7 RTM and Lower"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_WINDOWSUPDATEAGENTVERSION on SMS_G_System_WINDOWSUPDATEAGENTVERSION.ResourceID = SMS_R_System.ResourceId inner join SMS_G_System_OPERATING_SYSTEM on SMS_G_System_OPERATING_SYSTEM.ResourceID = SMS_R_System.ResourceId where SMS_G_System_WINDOWSUPDATEAGENTVERSION.Version < '7.6.7600.256' and SMS_G_System_OPERATING_SYSTEM.Version <= '6.1.7600'";Comment = "All systems with windows update agent with outdated version Win7 RTM and lower"; Foldername = "Operational"}
    ,@{Name = "Windows Update Agent | Outdated Version Win7 SP1 and Higher"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_WINDOWSUPDATEAGENTVERSION on SMS_G_System_WINDOWSUPDATEAGENTVERSION.ResourceID = SMS_R_System.ResourceId inner join SMS_G_System_OPERATING_SYSTEM on SMS_G_System_OPERATING_SYSTEM.ResourceID = SMS_R_System.ResourceId where SMS_G_System_WINDOWSUPDATEAGENTVERSION.Version < '7.6.7600.320' and SMS_G_System_OPERATING_SYSTEM.Version >= '6.1.7601'";Comment = "All systems with windows update agent with outdated version Win7 SP1 and higher"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Android"; Query = "SELECT SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client FROM SMS_R_System INNER JOIN SMS_G_System_DEVICE_OSINFORMATION ON SMS_G_System_DEVICE_OSINFORMATION.ResourceID = SMS_R_System.ResourceId WHERE SMS_G_System_DEVICE_OSINFORMATION.Platform like 'Android%'";Comment = "All Android modible devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Iphone"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_DEVICE_COMPUTERSYSTEM on SMS_G_System_DEVICE_COMPUTERSYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_DEVICE_COMPUTERSYSTEM.DeviceModel like '%Iphone%'";Comment = "All Iphone modible devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Ipad"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_DEVICE_COMPUTERSYSTEM on SMS_G_System_DEVICE_COMPUTERSYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_DEVICE_COMPUTERSYSTEM.DeviceModel like '%Ipad%'";Comment = "All Ipad mobile devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Windows Phone 8"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_DEVICE_OSINFORMATION on SMS_G_System_DEVICE_OSINFORMATION.ResourceID = SMS_R_System.ResourceId where SMS_G_System_DEVICE_OSINFORMATION.Platform = 'Windows Phone' and SMS_G_System_DEVICE_OSINFORMATION.Version like '8.0%'";Comment = "All Windows 8 mobile devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Windows Phone 8.1"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_DEVICE_OSINFORMATION on SMS_G_System_DEVICE_OSINFORMATION.ResourceID = SMS_R_System.ResourceId where SMS_G_System_DEVICE_OSINFORMATION.Platform = 'Windows Phone' and SMS_G_System_DEVICE_OSINFORMATION.Version like '8.1%'";Comment = "All Windows 8.1 mobile devices"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Microsoft Surface"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Model like '%Surface%'";Comment = "All Windows RT mobile devices"; Foldername = "Operational"}
    ,@{Name = "System Health | Disabled"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.UserAccountControl ='4098'";Comment = "All systems with client state disabled"; Foldername = "Operational"}
    ,@{Name = "Systems | x86"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceID = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.SystemType = 'X86-based PC'";Comment = "All systems with 32-bit system type"; Foldername = "Operational"}
    ,@{Name = "Systems | x64"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceID = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.SystemType = 'X64-based PC'";Comment = "All systems with 64-bit system type"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU1"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1203'";Comment = "All systems with SCCM client version R2 CU1 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU2"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1303'";Comment = "All systems with SCCM client version R2 CU2 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU3"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion like '5.00.7958.14%'";Comment = "All systems with SCCM client version R2 CU3 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU4"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1501'";Comment = "All systems with SCCM client version R2 CU4 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU5"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1604'";Comment = "All systems with SCCM client version R2 CU5 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 CU0"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.7958.1000'";Comment = "All systems with SCCM client version R2 CU0 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 SP1"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8239.1000'";Comment = "All systems with SCCM client version R2 SP1 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 SP1 CU1"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8239.1203'";Comment = "All systems with SCCM client version R2 SP1 CU1 installed"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Workstation 10.0%'";Comment = "All workstations with Windows 10 operating system"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 SP1 CU2"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8239.1301'";Comment = "All systems with SCCM client version R2 SP1 CU2 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | 1511"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8325.1000'";Comment = "All systems with SCCM client version 1511 installed"; Foldername = "Operational"}
    ,@{Name = "Laptops | Dell"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Manufacturer like '%Dell%'";Comment = "All laptops with Dell manufacturer"; Foldername = "Operational"}
    ,@{Name = "Laptops | Lenovo"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Manufacturer like '%Lenovo%'";Comment = "All laptops with Lenovo manufacturer"; Foldername = "Operational"}
    ,@{Name = "Laptops | HP"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Manufacturer like '%HP%' or SMS_G_System_COMPUTER_SYSTEM.Manufacturer like '%Hewlett-Packard%'";Comment = "All laptops with HP manufacturer"; Foldername = "Operational"}
    ,@{Name = "Clients Version | R2 SP1 CU3"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8239.1403'";Comment = "All systems with SCCM client version R2 SP1 CU3 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | 1602"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion = '5.00.8355.1000'";Comment = "All systems with SCCM client version 1602 installed"; Foldername = "Operational"}
    ,@{Name = "Clients Version | 1606"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion like '5.00.8412.100%'";Comment = "All systems with SCCM client version 1606 installed"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Microsoft Surface 3"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Model = 'Surface Pro 3' OR SMS_G_System_COMPUTER_SYSTEM.Model = 'Surface 3'";Comment = "All Microsoft Surface 3"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Microsoft Surface 4"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_COMPUTER_SYSTEM on SMS_G_System_COMPUTER_SYSTEM.ResourceId = SMS_R_System.ResourceId where SMS_G_System_COMPUTER_SYSTEM.Model = 'Surface Pro 4'";Comment = "All Microsoft Surface 4"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | Windows Phone 10"; Query = "select SMS_R_System.ResourceId, SMS_R_System.ResourceType, SMS_R_System.Name, SMS_R_System.SMSUniqueIdentifier, SMS_R_System.ResourceDomainORWorkgroup, SMS_R_System.Client from SMS_R_System inner join SMS_G_System_DEVICE_OSINFORMATION on SMS_G_System_DEVICE_OSINFORMATION.ResourceID = SMS_R_System.ResourceId where SMS_G_System_DEVICE_OSINFORMATION.Platform = 'Windows Phone' and SMS_G_System_DEVICE_OSINFORMATION.Version like '10%'";Comment = "All Windows Phone 10"; Foldername = "Operational"}
    ,@{Name = "Mobile Devices | All"; Query = "select * from SMS_R_System where SMS_R_System.ClientType = 3";Comment = "All Mobile Devices"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 v1507"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Build like '10.0.10240%'";Comment = "All workstations with Windows 10 operating system v1507"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 v1511"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Build like '10.0.10586%'";Comment = "All workstations with Windows 10 operating system v1511"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 v1607"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.Build like '10.0.14393%'";Comment = "All workstations with Windows 10 operating system v1607"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Current Branch (CB)"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.OperatingSystemNameandVersion like '%Workstation 10.0%' and SMS_R_System.OSBranch = '0'";Comment = "All workstations with Windows 10 CB"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Current Branch for Business (CBB)"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.OperatingSystemNameandVersion like '%Workstation 10.0%' and SMS_R_System.OSBranch = '1'";Comment = "All workstations with Windows 10 CBB"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Long Term Servicing Branch (LTSB)"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.OperatingSystemNameandVersion like '%Workstation 10.0%' and SMS_R_System.OSBranch = '2'";Comment = "All workstations with Windows 10 LTSB"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Support State - Current"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System  LEFT OUTER JOIN SMS_WindowsServicingStates ON SMS_WindowsServicingStates.Build = SMS_R_System.build01 AND SMS_WindowsServicingStates.branch = SMS_R_System.osbranch01 where SMS_WindowsServicingStates.State = '2'";Comment = "Windows 10 Support State - Current"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Support State - Expired Soon"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System  LEFT OUTER JOIN SMS_WindowsServicingStates ON SMS_WindowsServicingStates.Build = SMS_R_System.build01 AND SMS_WindowsServicingStates.branch = SMS_R_System.osbranch01 where SMS_WindowsServicingStates.State = '3'";Comment = "Windows 10 Support State - Expired Soon"; Foldername = "Operational"}
    ,@{Name = "Workstations | Windows 10 Support State - Expired"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System  LEFT OUTER JOIN SMS_WindowsServicingStates ON SMS_WindowsServicingStates.Build = SMS_R_System.build01 AND SMS_WindowsServicingStates.branch = SMS_R_System.osbranch01 where SMS_WindowsServicingStates.State = '4'";Comment = "Windows 10 Support State - Expired"; Foldername = "Operational"}
    ,@{Name = "Servers | Windows 2016"; Query = "select SMS_R_System.ResourceID,SMS_R_System.ResourceType,SMS_R_System.Name,SMS_R_System.SMSUniqueIdentifier,SMS_R_System.ResourceDomainORWorkgroup,SMS_R_System.Client from SMS_R_System where OperatingSystemNameandVersion like '%Server 10%'";Comment = "All Servers with Windows 2016"; Foldername = "Operational"}
    ,@{Name = "Clients Version | 1610"; Query = "select SMS_R_SYSTEM.ResourceID,SMS_R_SYSTEM.ResourceType,SMS_R_SYSTEM.Name,SMS_R_SYSTEM.SMSUniqueIdentifier,SMS_R_SYSTEM.ResourceDomainORWorkgroup,SMS_R_SYSTEM.Client from SMS_R_System where SMS_R_System.ClientVersion like '5.00.8458.100%'";Comment = "All systems with SCCM client version 1610 installed"; Foldername = "Operational"}
    ,@{Name = "Windows 10 1511 OSD"; Query = ""; Comment = "Windows 10 1511 OSD" ;Foldername = "Operating Systems"}
  )

  #Create Collection
  foreach ($Collection in $Arrcollection)
    {
    New-CMDeviceCollection -Name $Collection.Name -Comment $Collection.Comment -LimitingCollectionName $LimitingCollection -RefreshSchedule $Schedule -RefreshType 2 | Out-Null
    try
        {
        Add-CMDeviceCollectionQueryMembershipRule -CollectionName $Collection.Name -QueryExpression $Collection.Query -RuleName $Collection.Name
        Write-host *** Collection $Collection.Name created ***
        }
    catch
        {}
    #Move the collection to the right folder
    $FolderPath = "CBL:\DeviceCollection\" + $Collection.FolderName
    Move-CMObject -FolderPath $FolderPath -InputObject (Get-CMDeviceCollection -Name $Collection.Name)
    }
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {

  Install-WindowsFeature FS-DFS-Namespace, FS-DFS-Replication, RSAT-DFS-Mgmt-Con

  New-Item -ItemType 'directory' -Path 'D:\dfsroot'
   
  Set-Location D:\dfsroot

  New-Item -ItemType 'directory' -Path 'Application Management'
  New-Item -ItemType 'directory' -Path 'Compliance Settings'
  New-Item -ItemType 'directory' -Path 'Export'
  New-Item -ItemType 'directory' -Path 'Import'
  New-Item -ItemType 'directory' -Path 'Logs'
  New-Item -ItemType 'directory' -Path 'Logs\Archived Logs'
  New-Item -ItemType 'directory' -Path 'Logs\Task Sequence Error Logs'
  New-Item -ItemType 'directory' -Path 'Operating Systems'
  New-Item -ItemType 'directory' -Path 'Resources'
  New-Item -ItemType 'directory' -Path 'Resources\tools'
  New-Item -ItemType 'directory' -Path 'Resources\scripts'
  New-Item -ItemType 'directory' -Path 'Resources\hotfixes'
  New-Item -ItemType 'directory' -Path 'Resources\CMCurrent-Branch Updates'
  New-Item -ItemType 'directory' -Path 'Software Updates'


  New-Item -ItemType 'directory' -Path 'Application Management\Packages\Adobe'
  New-Item -ItemType 'directory' -Path 'Application Management\Packages\IBM'
  New-Item -ItemType 'directory' -Path 'Application Management\Packages\In-House'
  New-Item -ItemType 'directory' -Path 'Application Management\Packages\Microsoft' 
  New-Item -ItemType 'directory' -Path 'Application Management\Packages\Other'

  New-Item -ItemType 'directory' -Path 'Application Management\Applications\Adobe'
  New-Item -ItemType 'directory' -Path 'Application Management\Applications\IBM'
  New-Item -ItemType 'directory' -Path 'Application Management\Applications\In-House'
  New-Item -ItemType 'directory' -Path 'Application Management\Applications\Microsoft' 
  New-Item -ItemType 'directory' -Path 'Application Management\Applications\Other'

  New-Item -ItemType 'directory' -Path 'Export\Task Sequences Monitor'
  New-Item -ItemType 'directory' -Path 'Import\Applications'
  New-Item -ItemType 'directory' -Path 'Import\Baselines'
  New-Item -ItemType 'directory' -Path 'Import\Hardware Inventory MOFs'
  New-Item -ItemType 'directory' -Path 'Import\Report Definitions'
  New-Item -ItemType 'directory' -Path 'Import\Tasksequences'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Boot Images'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Branding'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Captures'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Task Sequence Media'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Virtual Hard Disks'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Scripts'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Scripts\Supported Models'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Scripts\Supported Models\SetDriverCategory'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Scripts\Supported Models\SetHWApps'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages\Dell'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages\HP'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages\Lenovo'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Driver Packages\Microsoft'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers\Dell'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers\HP'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers\Lenovo'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Drivers\Microsoft'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows 7'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows 8.1'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows 10'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows Windows Server 2012 R2'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Images\Windows Server 2016'

  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows 7'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows 8.1'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows 10'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows Windows Server 2012 R2'
  New-Item -ItemType 'directory' -Path 'Operating Systems\Operating System Upgrade Packages\Windows Server 2016'

  New-Item -ItemType 'directory' -Path 'Software Updates\Deployment Packages'
  New-Item -ItemType 'directory' -Path 'Software Updates\SCUP'
  New-Item -ItemType 'directory' -Path 'Software Updates\ExportSynchronizedProducts'

}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  Import-Module $env:SMS_ADMIN_UI_PATH.Replace("\bin\i386","\bin\configurationmanager.psd1") -force                       
  $SiteCode = Get-PSDrive -PSProvider CMSITE                                                                        

  Set-Location ($SiteCode.Name +":")
  #$pwd = read-host -AsSecureString

  #create new credential variable for CMInstall
  $NAA = "Training\SCCM_NAA"
  $pwd = ConvertTo-SecureString "T0psecret" -AsPlainText -Force

  New-CMAccount -Name $NAA -Password $pwd -Sitecode $SiteCode.Name

  #make the created user account your new Network Access Account

  $component = gwmi -class SMS_SCI_ClientComp -Namespace "root\sms\site_$($SiteCode.name)"  | Where-Object {$_.ItemName -eq "Software Distribution"}
  $component
  $props = $component.PropLists

  $prop = $props | where {$_.PropertyListName -eq "Network Access User Names"}

  $new = [WmiClass] "root\sms\site_$($SiteCode.name):SMS_EmbeddedPropertyList"
  $embeddedproperylist = $new.CreateInstance()

  $embeddedproperylist.PropertyListName = "Network Access User Names"
  $embeddedproperylist.Values = $NAA
  $component.PropLists += $embeddedproperylist
  $component.Put() | Out-Null

  # Configure boundary + boundarygroup

  $BGName = "Labo"
  New-CMBoundary -type IPRange -Value "172.30.150.1-172.30.150.254" -name "Trainig IP range"
  New-CMBoundaryGroup -Name $BGName
  Add-CMBoundaryToGroup -Boundaryname "Trainig IP range" -BoundaryGroupName $BGName
  Set-CMDistributionPoint -SiteSystemServerName "Primary.Training.Local" -AddBoundaryGroupName $BGName

}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname)
  $lastsccmbuild = get-wmiobject -namespace root\sms\site_cbl -class SMS_CM_Updatepackages | Select-Object fullversion,state | Sort-Object fullversion -Descending
  if ($lastsccmbuild[0].state -eq "262146")
    {write-host "Last build available... Starting the update"
    $filter = "Fullversion='" + $lastsccmbuild[0].fullversion +"'"
    $Update = get-wmiobject -namespace root\sms\site_cbl -class SMS_CM_Updatepackages -filter $filter  
    $Update.UpdatePrereqAndStateFlags(0,2)
    }
  else
    {write-error "Last $lastsccmbuild not available yet"
    }
}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```