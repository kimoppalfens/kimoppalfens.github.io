---
title: "Setting up an SCCM Labo using Powershell Direct - Part 2"
header:
  teaser: PowerShell512x384.png
author: Tom Degreef
date: 2017-03-25
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

Automating a full SCCM Labo setup using PowerShell Direct.
Part 2 will be on setting up and configuring the Domain Controller.

# Intro #

Just like in [Part 1 - Setting up a Router VM](http://www.oscc.be/sccm/configmgr/powershell/powershell%20direct/Setting-up-an-SCCM-Labo-using-Powershell-Direct-Part-1), I'll break down the script into several blocks to explain what each part does.
The beginning of the script is identical to setting up the router VM and also any following virtual machine in next posts, so I won't go into to much detail there.

## The Script ##

The Pre-requisites for the script to run successfully :

1. A sysprepped VHDX with Server 2016 installed
2. The SCCM 1606 ISO (to be able to extend the AD schema)

## Step 1 ##

```posh
$VMName = "DC"
$Memory = 2GB
$NewVHD = "D:\VM\VHD\" +$VMName + ".vhdx"
$VHDSize = "100000000000"
$Switchexternal = "externalSwitch"
$Switchprivate = "privateSwitch"
$Cpu=1
$IP = "172.30.150.1"
$Localadminpwd = "MyHyper$3curePwd!"
$Domainadminpwd = "MyHyper$3cureD0ma!nPwd!"
$UserPwd = "U$3rPwd!"
$domainname = "MyDomain.local" 
$netbiosName = "MyDomain"

function Test-IsVMUP ($VmName, $Argumentlist, $Credential)
{
    start-sleep -seconds 30
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
      write-host "failed, retrying..."
      Start-Sleep -seconds 2
    }
  }
  until ($x -eq $true) 
}

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
      write-host "Process still running, retrying..."
      Start-Sleep -seconds 15
    }
  }
  until ($x -eq $true) 
}

Copy-Item -Path D:\Sources\2016Sysprep.vhdx  -Destination $NewVHD

NEW-VM –Name $Vmname –MemoryStartupBytes $Memory -VHDPath $NewVHD -Generation 2 –Switchname $Switchprivate
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
  Rename-NetAdapter -name $first.Name -NewName "Private"
  Get-NetAdapter -Name "Private"
}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname,$myip)
  New-NetIPAddress -InterfaceAlias "Private" -IPAddress $myip -DefaultGateway 172.30.150.254 -PrefixLength 24 
  Set-DnsClientServerAddress -InterfaceAlias "Private" -ServerAddresses $myip
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$ip

$script = {
  if ((Get-DnsClientServerAddress -InterfaceAlias "Private" | ?{$_.AddressFamily -eq '2'} ).ServerAddresses[0] -ne "172.30.150.1")
  {
    Write-Error "First dns server not set correctly"
  }
}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```
As you can see this looks very identical to the Router script. Four new variables have been added :
- DomainadminPwd -> The domain admin password of your new domain
- DomainName -> the actual FQDN of your new domain
- NetbiosName  -> the netbios version of your domainname
- UserPwd -> we are going to add a few users to our domain and give them all the same password (This is lab environment anyway)

We did add a second function (LoopInvoke-Command) to test if a certain script-block has finished running. This works very similar to the function on testing if a Virtual machine is up and running. We use this new function to stop the "main" script from going forward after a command has been launched that "releases control" immediately. As a result the next command would execute already while the initial one is still running.

The last Scriptblock here was introduced to verify that the modifications to the network adapter were properly set.

## Step 2 ##

```posh
$script = {
  param($remotevmname)
  Install-WindowsFeature DNS –IncludeManagementTools
  Install-Windowsfeature DHCP -IncludeManagementTools
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname,$domainpwd,$domname,$netbname)
  Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
  $passwd = ConvertTo-SecureString $domainpwd -AsPlainText -Force
  Import-Module ADDSDeployment 
  Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\Windows\NTDS" -DomainMode "Win2012R2" -DomainName $domname -DomainNetbiosName $netbname -ForestMode "Win2012R2" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\Windows\SYSVOL" -Force:$true -SafeModeAdministratorPassword $passwd
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$Domainadminpwd,$domainname,$netbiosName

```

The first scriptblock just adds the DNS and DHCP Windows features as these are pre-requisites to install a domain controller.
The second block is the actual installation/configuration of the domain controller with the parameters from the initial step.
We convert domain password to a secure string, import the module needed to configure the domain and finally install a new forest.

## Step 3 ##

```Posh
$password = ConvertTo-SecureString $Domainadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("$netbiosName\administrator", $password )

$script = {
  if  ((get-eventlog -LogName "dns server" | ? {$_.EventID -eq "4500"}).Count -gt 0 )
  {
    Write-Host "Dns server registered in Application Configuration partition, assuming domain controller is ready"
  }    
  else
  {
    Write-Error "Waiting for Domain configuration to finalize"
  }

}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

```
The rest of the script will be running under the new domain admin credentials. So first thing we need to do is to convert this again to secure credentials.
Setting up the actual domain requires some time. We are querying the DNS Server eventlog for event 4500 as a "proof" that the initial DC configuration has finished.

## Step 4 ##

```Posh
$script = {
  param($remotevmname,$myip,$domname)
  import-module DhcpServer
  $FQDN = $remotevmname + "." + $domname
  set-dhcpserverV4binding -interfacealias "private" -BindingState $True 
  Add-DhcpServerInDC -DnsName $FQDN -IPAddress $myip
  add-dhcpserverv4scope -startrange 172.30.150.100 -endrange 172.30.150.150 -subnetmask 255.255.255.0 -name "Training Clients" -state active
  Set-DhcpServerv4OptionValue -ScopeId 172.30.150.0 -Router 172.30.150.254
  Set-DhcpServerv4OptionValue -DnsServer $myip -DnsDomain $domname
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$IP,$domainname
```

Next step is to configure the DHCP server so we can hand out IP addresses to our clients.
I think most steps are pretty clear but let me know if it isn't ;-)
This step could use some cleaning up in deriving the DHCP settings from the DC's IP but that might be for v2 of the scripts.

## Step 5 ##

```Posh
$script = {
  param($remotevmname,$userpassword)
  Import-Module ActiveDirectory
  New-ADUser -SamAccountName CMInstall -Name "CMInstall" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False
  New-ADUser -SamAccountName SCCM_NAA -Name "SCCM_NAA" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False
  New-ADUser -SamAccountName SCCM_DomainJoin -Name "SCCM_DomainJoin" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False
  New-ADUser -SamAccountName SCCM_ErrorLog -Name "SCCM_ErrorLog" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False
  New-ADUser -SamAccountName SCCM_Reporting -Name "SCCM_Reporting" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False

  New-ADOrganizationalUnit -Name "Corp Computers" -path "DC=Training,DC=Local" 
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$userpwd

#Extend Schema on DC
Add-VMDvdDrive -VmName $vmname
Set-VMDvdDrive -VMname $vmname -Path D:\Sources\mu_system_center_configuration_manager_endpoint_protection_version_1606_x86_x64_dvd_9678361.iso

$script = {
  param($remotevmname)
  D:\SMSSETUP\Bin\X64\Extadsch.exe
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

We continue with creating a few domain users that will be used later on in SCCM :
- CMInstall : used for setting up SCCM. Our SuperAdmin
- SCCM_NAA : The network Access Account in SCCM.
- SCCM_DomainJoin : This account will be configured so it can be used to join machines to the domain but only on a specific OU (see further down)
- SCCM_ErrorLog : Account used to centralize error logging in case a task sequence fails
- SCCM_Reporting : The acocunt needed to setup the reporting role in SCCM
Finally I add a new OU. This is the OU that SCCM_DomainJoin can use to join computers in the domain.

In order to extend the AD Schema for SCCM our VM Needs a virtual DVD Drive. Once that is added we mount the SCCM 1606 DVD so we can access it in our VM.

Extending the AD Schema is a simple as running "ExtADSch" from the SCCM DVD with a AD Schema admin user. (our Domain admin in this case)

## Step 6 ##

```Posh
$script = {
  param($remotevmname)
  Import-Module ActiveDirectory
  New-ADObject -Type Container -name "System Management" -Path "CN=System,DC=Training,DC=Local" -Passthru 
  $path = "AD:Cn=System Management,CN=System,DC=Training,DC=Local"
  $acl = Get-Acl -Path $path 
  $group = new-ADgroup CMServers -GroupScope Global -PassThru 
  $Sid = New-Object System.Security.Principal.SecurityIdentifier $group.sid 
  $nullGUID = [guid]'00000000-0000-0000-0000-000000000000' 
  $arg = $sid, "GenericAll","Allow", "All",$nullGUID 
  $ace = New-Object System.directoryservices.ActiveDirectoryAccessRule $arg 
  $acl.AddAccessRule($ace) 
  Set-Acl -Path $path -AclObject $acl 
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname
```

We're nearly there ! To allow SCCM to publish data to SCCM we need the "System Management" Container.
This block does that. Once the actual container has been created using the "New-ADObject" commandlet, I create an ad group called "CMServers". Once my SCCM Server is set up, I can add it to this group and it will have the appropriate rights to publish in this container.
Setting the appropriate rights was a challenge and took a lot of fidling. Eventually I succeeded with a combination of multiple blogposts found on the internet (that I can't recall the source of anymore)

## Step 7 ##

```Posh
$script = {
  [CmdletBinding(SupportsShouldProcess=$true)]
  param(
    [parameter(Position=0,Mandatory=$false)]
    [Security.Principal.NTAccount] $Identity = 'SCCM_DomainJoin',
    [parameter(Position=1,Mandatory=$false,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [alias("ComputerName")]
    [String[]] $Name = "Corp Computers",
    [String] $Domain,
    [String] $Server,
    [Management.Automation.PSCredential] $Credential
  )

  begin {
    Import-Module ActiveDirectory
    #Bring up an Active Directory command prompt so we can use this later on in the script
    cd ad:
    #Get a reference to the RootDSE of the current domain
    $rootdse = Get-ADRootDSE
    #Get a reference to the current domain
    $domain = Get-ADDomain

    #Create a hashtable to store the GUID value of each schema class and attribute
    $guidmap = @{}
    Get-ADObject -SearchBase ($rootdse.SchemaNamingContext) -LDAPFilter `
    "(schemaidguid=*)" -Properties lDAPDisplayName,schemaIDGUID | 
    % {$guidmap[$_.lDAPDisplayName]=[System.GUID]$_.schemaIDGUID}

    #Create a hashtable to store the GUID value of each extended right in the forest
    $extendedrightsmap = @{}
    Get-ADObject -SearchBase ($rootdse.ConfigurationNamingContext) -LDAPFilter `
    "(&(objectclass=controlAccessRight)(rightsguid=*))" -Properties displayName,rightsGuid | 
    % {$extendedrightsmap[$_.displayName]=[System.GUID]$_.rightsGuid}
    # Validate if identity exists
    try {
      [Void] $identity.Translate([Security.Principal.SecurityIdentifier])
    }
    catch [Security.Principal.IdentityNotMappedException] {
      throw "Unable to identify identity - '$identity'"
    }

    # Create DirectorySearcher object
    $Searcher = [ADSISearcher] ""

    # Initializes DirectorySearcher object
    function Initialize-DirectorySearcher {
      [Void] $Searcher.PropertiesToLoad.Add("distinguishedName")
      if ( $Domain ) {
        if ( $Server ) {
          $path = "LDAP://$Server/$Domain"
        }
        else {
          $path = "LDAP://$Domain"
        }
      }
      else {
        if ( $Server ) {
          $path = "LDAP://$Server"
        }
        else {
          $path = ""
        }
      }
      if ( $Credential ) {
        $networkCredential = $Credential.GetNetworkCredential()
        $dirEntry = New-Object DirectoryServices.DirectoryEntry(
          $path,
          $networkCredential.UserName,
          $networkCredential.Password
        )
      }
      else {
        $dirEntry = [ADSI] $path
      }
      $Searcher.SearchRoot = $dirEntry
      $Searcher.Filter = "(objectClass=domain)"
      try {
        [Void] $Searcher.FindOne()
      }
      catch [Management.Automation.MethodInvocationException] {
        throw $_.Exception.InnerException
      }
    }

    Initialize-DirectorySearcher

    # AD rights GUIDs
    $AD_RIGHTS_GUID_RESET_PASSWORD      = "00299570-246D-11D0-A768-00AA006E0529"
    $AD_RIGHTS_GUID_VALIDATED_WRITE_DNS = "72E39547-7B18-11D1-ADEF-00C04FD8D5CD"
    $AD_RIGHTS_GUID_VALIDATED_WRITE_SPN = "F3A64788-5306-11D1-A9C5-0000F80367C1"
    #$AD_RIGHTS_GUID_ACCT_RESTRICTIONS   = "4C164200-20C0-11D0-A768-00AA006E0529"
    $AD_RIGHTS_GUID_CHANGE_PASSWORD = "ab721a53-1e2f-11d0-9819-00aa0040529b"

    # Searches for a computer object; if found, returns its DirectoryEntry
    function Get-ComputerDirectoryEntry {
      param(
        [String] $name
      )
      #$Searcher.Filter = "(&(objectClass=computer)(name=$name))"
      $Searcher.Filter = "(&(objectClass=organizationalUnit)(name=$name))"
      try {
        $searchResult = $Searcher.FindOne()
        if ( $searchResult ) {
          $searchResult.GetDirectoryEntry()
        }
      }
      catch [Management.Automation.MethodInvocationException] {
        Write-Error -Exception $_.Exception.InnerException
      }
    }

    function Grant-ComputerJoinPermission {
      param(
        [String] $name
      )
      $domainName = $Searcher.SearchRoot.dc
      # Get computer DirectoryEntry
      $dirEntry = Get-ComputerDirectoryEntry "Corp Computers"
      if ( -not $dirEntry ) {
        Write-Error "Unable to find computer '$name' in domain '$domainName'" -Category ObjectNotFound
        return
      }
      if ( -not $PSCmdlet.ShouldProcess($name, "Allow '$identity' to join computer to domain '$domainName'") ) {
        return
      }
      # Build list of access control entries (ACEs)
      $accessControlEntries = New-Object Collections.ArrayList
      #--------------------------------------------------------------------------
      # Reset password
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ExtendedRightAccessRule(
            $identity,
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_RESET_PASSWORD,
            "Descendents",
            $guidmap["computer"]
          )
      ))

      #--------------------------------------------------------------------------
      # Change Password password
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ExtendedRightAccessRule(
            $identity,
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_CHANGE_PASSWORD,
            "Descendents",
            $guidmap["computer"]
          )
      ))
      #--------------------------------------------------------------------------
      # Validated write to DNS host name
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "Self",
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_VALIDATED_WRITE_DNS,
            "Descendents",
            $guidmap["computer"]

          )
      ))
      #--------------------------------------------------------------------------
      # Validated write to service principal name
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "Self",
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_VALIDATED_WRITE_SPN,
            "Descendents",
            $guidmap["computer"]
          )
      ))

      #--------------------------------------------------------------------------
      # Create & Delete Computer Objects
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "CreateChild,DeleteChild",
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $guidmap["computer"],
            "All"
          )
      ))
    
      #--------------------------------------------------------------------------
      # REad & Write all Computer Properties
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "ReadProperty,WriteProperty",
            [Security.AccessControl.AccessControlType] "Allow",
            "Descendents",
            [Guid] $guidmap["computer"]
          )
      ))
      `

      `


      #--------------------------------------------------------------------------
      # Modify Permission
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "ReadControl,WriteDACL",
            [Security.AccessControl.AccessControlType] "Allow",
            "Descendents",
            [Guid] $guidmap["computer"]
          )
      ))
      `
      `

      `


      #--------------------------------------------------------------------------
      # Write account restrictions
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "WriteProperty",
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_CHANGE_PASSWORD,
            "Descendents",
            $guidmap["computer"]
          )
      ))
      # Get ActiveDirectorySecurity object
      $adSecurity = $dirEntry.ObjectSecurity
      # Add ACEs to ActiveDirectorySecurity object
      $accessControlEntries | ForEach-Object {
        $adSecurity.AddAccessRule($_)
      }
      # Commit changes
      try {
        $dirEntry.CommitChanges()
      }
      catch [Management.Automation.MethodInvocationException] {
        Write-Error -Exception $_.Exception.InnerException
      }
    }
  }

  process {
    foreach ( $nameItem in $Name ) {
      Grant-ComputerJoinPermission $nameItem
    }
  }

}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList "SCCM_DomainJoin", "Corp Computers" 
```

The last block of code is taken from Bill Stewart's page. He wrote a script to grant Permissions to join the domain. ([Link](http://windowsitpro.com/windows-server/powershell-granting-computer-join-permissions))
We had to adopt it to work for our needs but truth to be told, Kim made those modifications as this was beyond my PowerShell knowledge. But the result is that we can grant account the appropriate permissions to an OU.

## Wrap up ##

You can find the full script here at the end of the blogpost.

Part 3 of this series will detail the actual setup and configuration of SCCM.

As usual, feel free to comment or let me know if something isn't clear for you or isn't working out at all.

```posh
$VMName = "DC"
$Memory = 2GB
$NewVHD = "D:\VM\VHD\" +$VMName + ".vhdx"
$VHDSize = "100000000000"
$Switchexternal = "externalSwitch"
$Switchprivate = "privateSwitch"
$Cpu=1
$IP = "172.30.150.1"
$Localadminpwd = "MyHyper$3curePwd!"
$Domainadminpwd = "MyHyper$3cureD0ma!nPwd!"
$UserPwd = "U$3rPwd!"
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
      write-host "Process still running, retrying..."
      Start-Sleep -seconds 15
    }
  }
  until ($x -eq $true) 
}


function Test-IsVMUP ($VmName, $Argumentlist, $Credential)
{
    start-sleep -seconds 30
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
      write-host "failed, retrying..."
      Start-Sleep -seconds 2
    }
  }
  until ($x -eq $true) 
}

Copy-Item -Path D:\Sources\2016Sysprep.vhdx  -Destination $NewVHD

NEW-VM -Name $Vmname -MemoryStartupBytes $Memory -VHDPath $NewVHD -Generation 2 -Switchname $Switchprivate
SET-VMProcessor -VMName $VMName -count $Cpu 
Set-VM -VMName $VMName -DynamicMemory -MemoryMaximumBytes $memory
start-vm $vmname

$password = ConvertTo-SecureString $Localadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("administrator", $password )

Test-IsVMUP -VmName $vmname -Argumentlist "Remote Registry" -Credential $cred

Stop-VM -Name $vmname
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
  Rename-NetAdapter -name $first.Name -NewName "Private"
  Get-NetAdapter -Name "Private"
}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  param($remotevmname,$myip)
  New-NetIPAddress -InterfaceAlias "Private" -IPAddress $myip -DefaultGateway 172.30.150.254 -PrefixLength 24 
  Set-DnsClientServerAddress -InterfaceAlias "Private" -ServerAddresses $myip
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$ip

$script = {
  if ((Get-DnsClientServerAddress -InterfaceAlias "Private" | ?{$_.AddressFamily -eq '2'} ).ServerAddresses[0] -ne "172.30.150.1")
  {
    Write-Error "First dns server not set correctly"
  }
}
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

#Install dns/DHCP

$script = {
  param($remotevmname)
  Install-WindowsFeature DNS -IncludeManagementTools
  Install-Windowsfeature DHCP -IncludeManagementTools
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname


#Setup DC

$script = {
  param($remotevmname,$domainpwd,$domname,$netbname)
  Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
  $passwd = ConvertTo-SecureString $domainpwd -AsPlainText -Force
  Import-Module ADDSDeployment 
  Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\Windows\NTDS" -DomainMode "Win2012R2" -DomainName $domname -DomainNetbiosName $netbname -ForestMode "Win2012R2" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\Windows\SYSVOL" -Force:$true -SafeModeAdministratorPassword $passwd
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$Domainadminpwd,$domainname,$netbiosName



#create new credential variable for domain admin
$password = ConvertTo-SecureString $Domainadminpwd -AsPlainText -Force
$cred= New-Object System.Management.Automation.PSCredential ("$netbiosName\administrator", $password )

#Wait for DC configuration to finalize
$script = {
  if  ((get-eventlog -LogName "dns server" | ? {$_.EventID -eq "4500"}).Count -gt 0 )
  {
    Write-Host "Dns server registered in Application Configuration partition, assuming domain controller is ready"
  }    
  else
  {
    Write-Error "Waiting for Domain configuration to finalize"
  }

}
#Start-Sleep -Seconds 400
LoopInvoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

#Setup DHCP
$script = {
  param($remotevmname,$myip,$domname)
  import-module DhcpServer
  $FQDN = $remotevmname + "." + $domname
  set-dhcpserverV4binding -interfacealias "private" -BindingState $True 
  Add-DhcpServerInDC -DnsName $FQDN -IPAddress $myip
  add-dhcpserverv4scope -startrange 172.30.150.100 -endrange 172.30.150.150 -subnetmask 255.255.255.0 -name "Training Clients" -state active
  Set-DhcpServerv4OptionValue -ScopeId 172.30.150.0 -Router 172.30.150.254
  Set-DhcpServerv4OptionValue -DnsServer $myip -DnsDomain $domname
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$IP,$domainname

#Create user CMInstall
$script = {
  param($remotevmname,$userpassword)
  Import-Module ActiveDirectory
  New-ADUser -SamAccountName CMInstall -Name "CMInstall" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False
  New-ADUser -SamAccountName SCCM_NAA -Name "SCCM_NAA" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False
  New-ADUser -SamAccountName SCCM_DomainJoin -Name "SCCM_DomainJoin" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False
  New-ADUser -SamAccountName SCCM_ErrorLog -Name "SCCM_ErrorLog" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False
  New-ADUser -SamAccountName SCCM_Reporting -Name "SCCM_Reporting" -AccountPassword (ConvertTo-SecureString -AsPlainText $userpassword -Force) -Enabled $true -PasswordNeverExpires $true -ChangePasswordATLogon $False

  New-ADOrganizationalUnit -Name "Corp Computers" -path "DC=Training,DC=Local" 
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname,$userpwd

#Extend Schema on DC
Add-VMDvdDrive -VmName $vmname
Set-VMDvdDrive -VMname $vmname -Path D:\Sources\mu_system_center_configuration_manager_endpoint_protection_version_1606_x86_x64_dvd_9678361.iso

$script = {
  param($remotevmname)
  D:\SMSSETUP\Bin\X64\Extadsch.exe
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

#Create system management container + AD Security group on that container

$script = {
  param($remotevmname)
  Import-Module ActiveDirectory
  New-ADObject -Type Container -name "System Management" -Path "CN=System,DC=Training,DC=Local" -Passthru 
  $path = "AD:Cn=System Management,CN=System,DC=Training,DC=Local"
  $acl = Get-Acl -Path $path 
  $group = new-ADgroup CMServers -GroupScope Global -PassThru 
  $Sid = New-Object System.Security.Principal.SecurityIdentifier $group.sid 
  $nullGUID = [guid]'00000000-0000-0000-0000-000000000000' 
  $arg = $sid, "GenericAll","Allow", "All",$nullGUID 
  $ace = New-Object System.directoryservices.ActiveDirectoryAccessRule $arg 
  $acl.AddAccessRule($ace) 
  Set-Acl -Path $path -AclObject $acl 
}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList $vmname

$script = {
  [CmdletBinding(SupportsShouldProcess=$true)]
  param(
    [parameter(Position=0,Mandatory=$false)]
    [Security.Principal.NTAccount] $Identity = 'SCCM_DomainJoin',
    [parameter(Position=1,Mandatory=$false,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [alias("ComputerName")]
    [String[]] $Name = "Corp Computers",
    [String] $Domain,
    [String] $Server,
    [Management.Automation.PSCredential] $Credential
  )

  begin {
    Import-Module ActiveDirectory
    #Bring up an Active Directory command prompt so we can use this later on in the script
    cd ad:
    #Get a reference to the RootDSE of the current domain
    $rootdse = Get-ADRootDSE
    #Get a reference to the current domain
    $domain = Get-ADDomain

    #Create a hashtable to store the GUID value of each schema class and attribute
    $guidmap = @{}
    Get-ADObject -SearchBase ($rootdse.SchemaNamingContext) -LDAPFilter `
    "(schemaidguid=*)" -Properties lDAPDisplayName,schemaIDGUID | 
    % {$guidmap[$_.lDAPDisplayName]=[System.GUID]$_.schemaIDGUID}

    #Create a hashtable to store the GUID value of each extended right in the forest
    $extendedrightsmap = @{}
    Get-ADObject -SearchBase ($rootdse.ConfigurationNamingContext) -LDAPFilter `
    "(&(objectclass=controlAccessRight)(rightsguid=*))" -Properties displayName,rightsGuid | 
    % {$extendedrightsmap[$_.displayName]=[System.GUID]$_.rightsGuid}
    # Validate if identity exists
    try {
      [Void] $identity.Translate([Security.Principal.SecurityIdentifier])
    }
    catch [Security.Principal.IdentityNotMappedException] {
      throw "Unable to identify identity - '$identity'"
    }

    # Create DirectorySearcher object
    $Searcher = [ADSISearcher] ""

    # Initializes DirectorySearcher object
    function Initialize-DirectorySearcher {
      [Void] $Searcher.PropertiesToLoad.Add("distinguishedName")
      if ( $Domain ) {
        if ( $Server ) {
          $path = "LDAP://$Server/$Domain"
        }
        else {
          $path = "LDAP://$Domain"
        }
      }
      else {
        if ( $Server ) {
          $path = "LDAP://$Server"
        }
        else {
          $path = ""
        }
      }
      if ( $Credential ) {
        $networkCredential = $Credential.GetNetworkCredential()
        $dirEntry = New-Object DirectoryServices.DirectoryEntry(
          $path,
          $networkCredential.UserName,
          $networkCredential.Password
        )
      }
      else {
        $dirEntry = [ADSI] $path
      }
      $Searcher.SearchRoot = $dirEntry
      $Searcher.Filter = "(objectClass=domain)"
      try {
        [Void] $Searcher.FindOne()
      }
      catch [Management.Automation.MethodInvocationException] {
        throw $_.Exception.InnerException
      }
    }

    Initialize-DirectorySearcher

    # AD rights GUIDs
    $AD_RIGHTS_GUID_RESET_PASSWORD      = "00299570-246D-11D0-A768-00AA006E0529"
    $AD_RIGHTS_GUID_VALIDATED_WRITE_DNS = "72E39547-7B18-11D1-ADEF-00C04FD8D5CD"
    $AD_RIGHTS_GUID_VALIDATED_WRITE_SPN = "F3A64788-5306-11D1-A9C5-0000F80367C1"
    #$AD_RIGHTS_GUID_ACCT_RESTRICTIONS   = "4C164200-20C0-11D0-A768-00AA006E0529"
    $AD_RIGHTS_GUID_CHANGE_PASSWORD = "ab721a53-1e2f-11d0-9819-00aa0040529b"

    # Searches for a computer object; if found, returns its DirectoryEntry
    function Get-ComputerDirectoryEntry {
      param(
        [String] $name
      )
      #$Searcher.Filter = "(&(objectClass=computer)(name=$name))"
      $Searcher.Filter = "(&(objectClass=organizationalUnit)(name=$name))"
      try {
        $searchResult = $Searcher.FindOne()
        if ( $searchResult ) {
          $searchResult.GetDirectoryEntry()
        }
      }
      catch [Management.Automation.MethodInvocationException] {
        Write-Error -Exception $_.Exception.InnerException
      }
    }

    function Grant-ComputerJoinPermission {
      param(
        [String] $name
      )
      $domainName = $Searcher.SearchRoot.dc
      # Get computer DirectoryEntry
      $dirEntry = Get-ComputerDirectoryEntry "Corp Computers"
      if ( -not $dirEntry ) {
        Write-Error "Unable to find computer '$name' in domain '$domainName'" -Category ObjectNotFound
        return
      }
      if ( -not $PSCmdlet.ShouldProcess($name, "Allow '$identity' to join computer to domain '$domainName'") ) {
        return
      }
      # Build list of access control entries (ACEs)
      $accessControlEntries = New-Object Collections.ArrayList
      #--------------------------------------------------------------------------
      # Reset password
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ExtendedRightAccessRule(
            $identity,
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_RESET_PASSWORD,
            "Descendents",
            $guidmap["computer"]
          )
      ))

      #--------------------------------------------------------------------------
      # Change Password password
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ExtendedRightAccessRule(
            $identity,
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_CHANGE_PASSWORD,
            "Descendents",
            $guidmap["computer"]
          )
      ))
      #--------------------------------------------------------------------------
      # Validated write to DNS host name
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "Self",
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_VALIDATED_WRITE_DNS,
            "Descendents",
            $guidmap["computer"]

          )
      ))
      #--------------------------------------------------------------------------
      # Validated write to service principal name
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "Self",
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_VALIDATED_WRITE_SPN,
            "Descendents",
            $guidmap["computer"]
          )
      ))

      #--------------------------------------------------------------------------
      # Create & Delete Computer Objects
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "CreateChild,DeleteChild",
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $guidmap["computer"],
            "All"
          )
      ))
    
      #--------------------------------------------------------------------------
      # REad & Write all Computer Properties
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "ReadProperty,WriteProperty",
            [Security.AccessControl.AccessControlType] "Allow",
            "Descendents",
            [Guid] $guidmap["computer"]
          )
      ))
      `

      `


      #--------------------------------------------------------------------------
      # Modify Permission
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "ReadControl,WriteDACL",
            [Security.AccessControl.AccessControlType] "Allow",
            "Descendents",
            [Guid] $guidmap["computer"]
          )
      ))
      `
      `

      `


      #--------------------------------------------------------------------------
      # Write account restrictions
      #--------------------------------------------------------------------------
      [Void] $accessControlEntries.Add((
          New-Object DirectoryServices.ActiveDirectoryAccessRule(
            $identity,
            [DirectoryServices.ActiveDirectoryRights] "WriteProperty",
            [Security.AccessControl.AccessControlType] "Allow",
            [Guid] $AD_RIGHTS_GUID_CHANGE_PASSWORD,
            "Descendents",
            $guidmap["computer"]
          )
      ))
      # Get ActiveDirectorySecurity object
      $adSecurity = $dirEntry.ObjectSecurity
      # Add ACEs to ActiveDirectorySecurity object
      $accessControlEntries | ForEach-Object {
        $adSecurity.AddAccessRule($_)
      }
      # Commit changes
      try {
        $dirEntry.CommitChanges()
      }
      catch [Management.Automation.MethodInvocationException] {
        Write-Error -Exception $_.Exception.InnerException
      }
    }
  }

  process {
    foreach ( $nameItem in $Name ) {
      Grant-ComputerJoinPermission $nameItem
    }
  }

}
Invoke-Command -VMName $vmname -credential $cred -ScriptBlock $script -ArgumentList "SCCM_DomainJoin", "Corp Computers" 
```