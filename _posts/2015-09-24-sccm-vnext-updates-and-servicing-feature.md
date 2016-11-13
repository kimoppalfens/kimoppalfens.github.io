---
title: "SCCM updates and servicing feature!"
author: Kim Oppalfens
date: 2015-09-24
categories:
  - SCCM
tags:
  - SCCM
  - Servicing
---

On the 23rd of September 2015 the ConfigMgr Product team released a [blog][1] on an update for Configuration Manager Technical Preview 3.

# Intro

This new and upcoming version of SCCM, is known by many as SCCM vNext. The blogpost contained roughly 321 words. (Yes I did put that in there to weasel you into going in and counting the words :) )  

The shortness of the blog, and it's number of words, in my opinion, doesn't do justice to the importance of this new feature called CM Updates, Updates and servicing or Easy setup, depending on who you talk to. This feature is the way forward for the product team to update the next release of Configuration Manager, and might well replace Service packs and/or CU's in the future. One of the challenges for the CM product team has always been to find the right "release vehicle". Release vehicles are a means to get updates out to the install base, and at present are usually one of these:  

* New release  
* Service pack  
* Cumulative Update  
* Web release/download  

The first 2 in that list, are at least perceived as disruptive to the systems management service, while Cumulative Updates have quite different perceptions across different IT departments and companies. There's definitely a set of IT departments that consider the CU's, and their installation disruptive as well. This means all sorts of things come into play when any of these are released, release management, a rollback plan or the change advisory board to name a few. The ambition of CM updates is to alleviate that pain through an integrated servicing mechanism much like other software that auto-updates, yet with admin approval.  

Or in the words of the blog: 

> "This update is the first to be delivered through our new "Updates and Servicing" node in the product. Moving forward, this is how you should expect to receive Technical Preview updates. When System Center Configuration Manager becomes generally available in Q4 of this calendar year, we will continue to use this same channel to provide a faster, lighter and easier update experience for new features, bug fixes, and more."  

People already involved with Microsoft Intune in a Hybrid scenario will see a lot of similarities with what is called the weave feature to download mobile device management extensions and enable/install these extensions. This should not come as a surprise, as one of the driving factors behind weave was the speed with which new versions and features of IOS, Android and Windows Phone were released. The mobile device management market is rapidly growing and evolving, and needed it's own release vehicle, that release vehicle became 'Extensions for Microsoft Intune' a.k.a weave. 

With Windows 10 entering the Windows as a service-era, with things like the current branch potentially upgrading at a rapid pace, Microsoft needed something similar to update general CM items, and not just mobile device management related items. The blog released yesterday was the announcement of "the very first real world public test of this mechanism!". The announcement itself lacked a bit of enthusiasm, but this is a feature that I am excited about for the future of CM.  

This feature can update and deliver fixes and features for:  

* Site servers  
* SMS_Provider  
* Configuration Manager console  
* Configuration manager clients  

Which should provide the necessary flexibility to update just about anything, as this allows the product team to update, the ConfigMgr File system, the Admin UI file system, the database (Data as well as new tables, views, Stored procedures,…, as WMI)  

The announcement itself didn't go into much detail explaining how the future worked, however did contain a [link][2] to the documentation of the Microsoft System Center Configuration Manager Technical Preview. This page contains a section on "Updates and servicing" which gives some insight into how this all operates.  

Some important items in that documentation, are highlighted below, but I do encourage you to go and read the doc itself to get all the nitty gritty details  

First of all, this entire process is driven by a new site system role called the cloud connection point, which replaces the Microsoft Intune connector site system role. In the current tech preview (Release 3), the cloud connection point needs to be installed on the primary site server and needs internet access. Both of these limitations are expected to be resolved in a subsequent release, and an offline procedure is planned for a future release, as mentioned in the comments section of the blog.  

Other limitations: Os language must be English, and you must set your date/time format to this absolutely weird way of throwing days, months and years in some random order called MM-DD-YYYY. Not sure who ever came up with that way of noting dates, but apparently that's what we need to use. In testing and hearing experiences from others, **this is the number one deal-breaker: The update process itself seems to be rock-solid, assuming you set your Date/time format accordingly.   
**

New updates are checked for every 7 days, since the install date of the environment. According to the docs a restart of the SMS_executive service triggers the check for updates as well. I'd venture a guess that restarting the SMS_DMP_Downloader might trick it as well, which would be less disruptive. (To be tested).  

Once the updates are downloaded you'll find them in **Administration** > **Cloud Services** > **Updates and Services** as available, you can subsequently click **Install Update Pack**

As a final note, just running the prerequisite checker standalone, from this same node, doesn't work in the current build. Triggering this will perform the install as well.  

1. Step 1 would be to restart your sms_executive service if the update hasn't arrive in your **Updates and Services** node.  

2. The update should arrive and be in the downloading state for a while. You can monitor the download progress in the DMPDownloader.log the log should contains lines similar to:  

```plaintext
EasySetupDownload thread is starting... $$<09-22-2015 21:21:36.981-120>
Download Easy setup payloads~~ $$<09-22-2015 21:21:37.008-120>
Get manifest.cab url~~ $$<09-22-2015 21:21:37.012-120>
Successfully write the update meta into outbox for package dcd17922-2c96-4bd7-b72d-e9159582cdf2~~ $$<09-22-2015 21:40:36.934-120>  
```

1. This particular update is somewhere between 800 and 900 Mbytes, so depending on your internet connection speed, it might take a while to download everything. In my particular experience, mumbling "patience is a virtue" over and over again had neither a positive nor negative impact on the download speed.  

2. Once downloaded you should see the update in your **EasySetupPayLoad**  

3. ![][3]![][4]  

4. If the update arrive your UI should look like the 2 screenshots below, and have the update state listed as Available.  

5. ![][5]![][6]  

6. Once the update's state switches to the available state, you can launch the "**Install Update Pack"** action from the Quick access toolbar, which provides you with the details of the CM Update and the ability to ignore prereq check warnings. (Feel free to leave this unchecked in the current build as warnings are ignored no matter what you select).  

7. ![][7]  

8. Next, you need to read the License agreement, and accept them. Yes, the idea is that you read them first.  

9. ![][8]  

10. Subsequently you can chose to upgrade all your clients at once, or perform testing to a pre-production collection you specify. This ties into the client updating feature that was enhanced in CM2012SP1CU1 to also support cumulative update releases as opposed to the original behavior which only supported service packs.  

11. ![][9]  

12. The rest of the wizard doesn't provide you with any options to configure, find screenshots below for completeness.  

13. ![][10]  

14. ![][11]  

15. ![][12]  

16. At this point the Administration node will list the Update in a state "installing". Depending on your environment this installation might take a while to complete.  

17. Logfiles involved during this upgrade are dmpdownloader.log, sitecomp.log and cmupdate.log and obviously the logs for prereq checking and setup which in my case are looked in the root of the c: drive.  

18. ![][13]  

19. In the file system you'll also notice the downloaded files are being copied to a brand new folder in your site server installation called CMUStaging  

20. Upon successful installation the **_MonitoringOverviewSite Servicing status_** should contain a status of installed  

21. And the about screen should contain a version of 1509, please note that the Console versions and Site version remain at 5.0.8299.1000  

22. ![][14]  

23. ![][15]  

  
 

Once the server is upgraded, the next time the admin ui is opened you'll be prompted by a message asking you to upgrade your admin UI to the newest version. Much like the request to enable new extensions for Intune when they have arrived. This is a similar system and poses the same challenges, you'll have to be an administrator and have the ability to install the upgrade for this to work successfully.  

1. ![][16]![][17]  

2. ![][18]  

3. Again this is a fairly regular install, so you can monitor progress by following the ConfigMgrAdminUISetup and its verbose variant.  

  
 

![][19]  

  
 

The process appears to be fairly solid in several tests I've ran, as long as you make sure you have the DateTime format set to MM-DD-YYYY, I know, it's an American thing, get over it. They'll come to their senses one day and adopt Metric, 220Volt and a sensible datetime format, and might even come up with a proper name for that sport where you're seldomly allowed to use your foot. (Don't hold your breath, for now).  

### Troubleshooting  

If you managed to break the upgrade anyway, have a look at the troubleshooting section in the link provide in the documentation section. But it roughly comes down to  

1. Make sure you set the datetime format correctly  

2. Are you 100% positive you verified item 1  

3. Really?  

4. If you goofed up on 1-3 run the following command in SQL Management studio, after typing a full page in Word with the sentence "I am a goof!"
  
```sql
EXEC spCMUSetUpdatePackageState N'dcd17922-2c96-4bd7-b72d-e9159582cdf2', 262146, N''
```

### Wmi  

5 new WMI classes have surfaced that seem to be related to this feature:  

* SMS_CM_UpdateFeatures (0 Instances, 1 method (UpdateFeatureExposureStatus)  
* SMS_CM_UpdatePackageFeatures (0 Instances, 1 method (UpdateFeatureExposureStatus)  
* SMS_CM_Update_Packages (1 instance with guid of update, 2 methods (IsCurrentWorkingUpdatePackage, updatePrereqAndStateFlags)  
* SMS_CM_UpdatePackageSiteStatus (1 instance with guid of update, no methods)  
* SMS_CM_UpdatePackDetailedSiteStatus (multiple instances with different steps ranging from prereq checking to actual install steps. (State 3, appears to be success (to be validated)  

### SQL  

5 new views where created related to the 5 WMI classes above:  

* Vsms_CM_updatefeatures  
* vSMS_CM_UpdatePackageFeatures  
* vSMS_CM_UpdatePackages  
* vSMS_CM_UpdatePackageSiteStatus  
* vSMS_CM_LatestInstalledPackageFeatures  

10 Stored procedures have surfaced that appear to be related to this new servicing feature  

![][20]  

20 new tables have surfaced that appear to be related to this new servicing feature  

![][21]  


[1]: http://blogs.technet.com/b/mniehaus/archive/2012/09/02/speed-up-mdt-task-sequences-in-configuration-manager.aspxhttp:/blogs.technet.com/b/configmgrteam/archive/2015/09/23/now-available-update-for-system-center-config-manager-tp3.aspx
[2]: http://blogs.technet.com/b/mniehaus/archive/2012/09/02/speed-up-mdt-task-sequences-in-configuration-manager.aspxhttps:/technet.microsoft.com/library/dn965439.aspx
[3]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup1.png
[4]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup2.png
[5]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup3.png
[6]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup4.png
[7]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup5.png
[8]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup6.png
[9]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup7.png
[10]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup8.png
[11]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup9.png
[12]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup10.png
[13]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup11.png
[14]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup12.png
[15]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup13.png
[16]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup14.png
[17]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup15.png
[18]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup16.png
[19]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup17.png
[20]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup18.png
[21]: http://scug.be/thewmiguy/files/2015/09/092415_1353_SCCMvNextup19.png
