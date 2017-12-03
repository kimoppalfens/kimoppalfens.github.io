---
title: "An interesting use-case for Intune and SCCM Co-Management - Part 3"
header:
author: Tom Degreef
date: 2017-11-27
categories:
  - SCCM
  - Configmgr
  - Intune
  - Co-Management
  - Cloud Management Gateway
  - CMG
  - Azure AD
  - AAD


tags:
  - SCCM
  - Configmgr
  - Intune
  - Co-Management
  - Cloud Management Gateway
  - CMG
---

Real-World scenario on where Intune and SCCM Co-management could come in handy

# Intro #

So we've had [Part 1](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/CoMGMT-usecase-Part-1/) for the Cloud Management Gateway.
And [Part 2](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/azure%20ad/aad/CoMGMT-usecase-Part-2/) for getting Azure AD Discovery up and running, now let's focus on configuring Co-Management itself !

## Setting up Co-management between Configmgr and Intune  ##

In your ConfigMgr Admin-UI, navigate to the Administration pane / Cloud Services / Co-Management and select "Configure Co-Management" from the quick access toolbar.

Sign in with your Intune Admin account and click next.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt1.PNG)

Select if you want to automatically enroll into Intune. If you enable this feature, all devices that are already managed by your on-premise ConfigMgr setup will automatically be enrolled into Intune.

Given that we are setting up Co-Management, the end-result would be that the device is both accessible for Intune & ConfigMgr. For the sake of this blog, let's put it on 'All'. However, keep in mind that a few pre-requisites on the client side need to be in place for this to work.

Those pre-requisites are :

- A windows 10 - 1709 (Fall creators update) machine
- The Configmgr agent updated to 1710 (5.00.8577.1003 or higher)
- a Valid Intune license 
- Multi Factor Authentication (eg Hello 4 Business)

We'll cover de technical details of these options later in this blog when we check out the result on an actual client.

Also, while you are on this screen, make sure to copy (and save!) the Command line you need later on if you want to install the Configmgr agent using Intune.  
If, for some reason, you don't see this command line and are greeted by a "Requirements not met" window, it probably means you missed something that we configured in Part 1 or 2.

Make sure to check back and verify each step.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt2.PNG)

For each of the 3 available workloads, configure if you want them managed by ConfigMgr, Intune, or start a pilot for Intune. Later on you can define a "Pilot" Collection.

Important to notice here is that there will be only 1 pilot collection available. This collection will be used for each workload you select on this screen and configured for Pilot.  
This Pilot collection will equally be used in the previous step if you chose "Pilot" instead of all for the automatic Intune enrollment.

Unless you are very familiar with each workload and how management difference between Intune and Configmgr, I would advise to start 1 workload at a time in Pilot. This allows you to evaluate each and every workload separately and doesn't overly complicates troubleshooting (if needed).  

In my particular case I will start with "Compliance Policies" in Pilot.  
In the follow-up blog posts we will continue with "Conditional Access" and that will explain more in depth why we started with the "Compliance Policies" workload here.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt3.PNG)

Select the Intune Pilot collection you are planning to use and click Next to continue the wizard.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt4.PNG)

Review your summary and finish the Wizard.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt5.PNG)

If, for some reason, you want to modify what entity manages what workload or, get that install Command line again or, change your pilot collection.  
Select "Properties" from the Quick access toolbar while you have your Co-management entry selected.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt6.PNG)

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt7.PNG)

That's it from a ConfigMgr point of view !

## An Additional pre-requisite ##

For Multi Factor Authentication (MFA). The [Microsoft docs](https://docs.microsoft.com/en-us/intune/windows-enroll#enable-windows-10-automatic-enrollment) mention it as "recommended" but they don't list it as required. 

However, from my own tests, I can conclude that without any form of MFA, I wasn't able to get auto-enrollment working at all.  
I would even get an error-message in the eventlog that specified "MFA Required".

For my specific tests, I setup a Hello for Business profile from within Configmgr and deployed that to my test-machine and enrolled in Hello for Business. It's beyond the scope of this blog to go into the details of setting up hello for business. But this, or any other form of MFA should be sufficient. 

## Verifying the results on a Windows 10 - 1709 Machine ##

Let's first verify that the Client Requirements are met.

Logon to your Windows 10 - 1709 client and make sure that it is Azure AD Registered.  
Do this by opening and Admin Command prompt and typing the following command : "DSREGCMD /Status".  
You should see something similar to this 

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_DR_succeeded.PNG)

Obviously, also make sure that your ConfigMgr client agent is up to the latest version (in production, it would need to be Configmgr 1710 or later)

If all of the pre-requisites are good, we can check if our enrollment in Intune has succeeded.  
There are a few places to check, but I found the CoManagementHandler.Log (under c:\windows\ccm\logs) to be the easiest.

**Note:**  
A few things I noticed about auto-enrollment is that once the policy is received by the client, it will not automatically start the enrollment process. There is some delay built-in to the mechanism that seems to randomize this from a few minutes to a few hours.

If enrollment fails (because not all requirements are met), it will retry 3 times with a 15 minute interval. If after that it still didn't succeed in enrolling, it will not try again until the computer is rebooted (or the sms agent host got restarted).

If, like me, you are a very patient person, you can sit out the time for the auto-enrollment to kick-in.  
However, if you want to speed things up, for the sake of testing, you can always trigger the enrollment yourself.  
The proper way to do this would be to open admin command-prompt (admin needs to be the same user that enrolled the device in Azure AD) and navigating to c:\windows\System32

Then run : "DeviceEnrollment /c /AutoEnrollMDM"

The command itself won't give any feedback but you should see some results in the Eventlog (Microsoft / Windows / DeviceManagement-Enterprise- Diagnostics-Provider / Admin).  
Look for EventID "75 - Auto MDM Enroll: Succeeded"

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt17.PNG)

If all goes well, your device should now be registered in Intune and still managed by ConfigMGr. The easiest to verify this is by logging into Intune and going to your devices node.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt15.PNG)

Click "All Devices" and you should see all registered devices in Intune. The device that just enrolled should be there and in the "Managed By" column, it should be marked as "MDM/ConfigMGr Agent"

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt16.PNG)

The CoManagementHandler.Log should show something similar to this :

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Comgmt18.PNG)

That's it for today ! Enjoy Reading.