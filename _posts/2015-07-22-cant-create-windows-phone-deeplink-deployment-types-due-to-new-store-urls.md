---
title: "Serious Windows 7 32bit software update problem"
author: Kim Oppalfens
date: 2015-03-30
categories:
  - SCCM
tags:
  - SCCM
  - Software Updates
---

A growing number of customers is contacting us about the issue going on below on their Windows 7 32 bit machines. I don't often ask people to distribute my blog information further. But quite a few customers should probably be warned for this issue.

**Problem description**

An issues exists at present where Windows 7 32 bit machines will reply compliant/installed on any software update they scan for, even the ones that aren't installed.

I have customers reporting updates failing to install because of this, and one where Cumulative Updates for ConfigMgr started reporting compliant without them creating a deployment for the updated client.

The problem can be seen at the client side in the Windowsupdate.log. When your log contains the following text "

```plaintext
GetWARNING: ISusInternal::GetUpdateMetadata2 failed, hr=8007000E
```

You're probably another victim of this terrible issue. The concern here is that a lot of environments might be unaware they have this issue, as nothing will point it out when looking at things centrally from the Admin UI. Clients will just report compliant on all their software update deployments.

**Identifying the problem in your environment**

The easiest way I could come up with to identify this problem in your environment is to create a configuration item to detect it. To do this:  

  

1. create a script configuration item. 

2. Select All Windows 7 32 bit as the supported platform  

3. Use String as the data type  

4. Choose powershell as your script language of choice  

5. Paste the following text in the discovery script:

```posh
select-string-pattern'GetWARNING: ISusInternal::GetUpdateMetadata2 failed, hr=8007000E'-path"$env:windirwindowsupdate.log" 
```  

6. Add the configuration item to a Configuration baseline 

7. Deploy the configuration baseline to All Windows 7 32bit machines  

8. The report list of assets by compliance state for a given baseline is a good report to check the results.  

9. !!!! **Any machines reporting compliant to this baseline have a serious issue as they won't install any software updates, yet report compliant on all !!!!**  
  
 

**Possible workarounds**  

  
  

    1. Decline unneeded updates within the WSUS server (Declined updates do not get offered to clients during scans.)  
  

        1. Unneeded updates include superseded updates, updates for products and/or classifications that are not present in the client environment, and expired updates.
  

        2. You can manually decline the updates within the WSUS console or use a script method . **NOTE:  Always backup the WSUS database (SUSDB) prior to performing any changes like this.**
  

        3. After declining unneeded updates, re-index the susdb, and run WSUS Server Cleanup Wizard:
  
  

* ****Set user VA to 3072 MB: **bcdedit /set IncreaseUserVA 3072 ******  
  

    1. This will free up another GB of memory in user space..
  

    2. This does require a restart of the machine.
  

    3. It's possible some machines or applications may have problems when this setting is enabled
  

1. Move wuauserv to its own SVCHost instance running following commands in elevated command prompt:  
  

    1. Net stop wuauserv
  

    2. 'sc config wuauserv type= own'
  

    3. Net start wuauserv
  
  

  
  
**More details:**

You can find the nitty gritty details and soulmates in this forum post.

https://social.technet.microsoft.com/Forums/en-US/cf8fbe28-714d-49d3-b2ce-5cc5f6f79c63/some-clients-not-updating-reporting-compliant-hr8007000e-error-in-windowsupdatelog?forum=configmanagersecurity
