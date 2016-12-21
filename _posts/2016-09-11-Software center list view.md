---
title: "Upon Request – Launching the CM Current branch Software center in List view"
author: Kim Oppalfens
header:
  overlay_image: logging1280x960.jpg
  teaser: logging512x384.jpg
date: 2016-09-13
categories:
  - Powershell
  - SCCM
tags:
  - SCCM
---

So this isn't going to be the cleanest blogpost, but I had 2 MVP's requesting the following functionality a short while ago.

# Intro #


Namely to be able to launch the new Configuration Manager current branch software center in List view. By default the new software center, aka scclient.exe lives in the ClientUX folder of your client installation path. The SCClient.exe however launches on the applications tab in what is referred to as the 'Tile View'

The script below will launch the tool, wait for a second, then send the keysequence to the "Software Center" window to switch it to List view.

PS: The Set foreground window part is code I found online somewhere, but can't recall where. If you see this as your code, credit where credit is due, contact me on twitter and I'll add a reference.


```posh
$ClientLogDirectory = (Get-CimInstance -Namespace root\ccm\policy\machine\actualconfig -ClassName ccm_logging_globalconfiguration).LogDirectory 
$ClientDir = $ClientLogDirectory.SUbstring(0,$ClientLogDirectory.Length-5) 
$scclient = $ClientDir+'\ClientUX\SCClient.exe' 
.$scclient 
Start-Sleep -Seconds 1 

# load assembly cotaining class System.Windows.Forms.SendKeys 
[void] [Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms') 

Add-Type -TypeDefinition @" 
 using System; 
 using System.Runtime.InteropServices; 
 public class StartActivateProgramClass { 
 [DllImport("user32.dll")] 
 [return: MarshalAs(UnmanagedType.Bool)] 
 public static extern bool SetForegroundWindow(IntPtr hWnd); 
 } 
"@ 

if (Get-Process | Where-Object -FilterScript {
    $_.MainWindowTitle -eq 'Software Center' 
}) 

{
  $h = $p[0].MainWindowHandle 

  [void] [StartActivateProgramClass]::SetForegroundWindow($h) 

  [System.Windows.Forms.SendKeys]::SendWait('{TAB}{TAB}{TAB}{TAB}{TAB}{TAB}{TAB}{TAB}{TAB}{LEFT} ')
} 
```

