---
title: "Technical Preview 1706 & Current Branch 1706 feature highlight : Android and iOS Enrollment Restrictions"
header:
author: Tom Degreef
date: 2017-08-15
categories:
  - SCCM
  - Configmgr
  - TP
  - Intune
  - hybrid

tags:
  - SCCM
  - Configmgr
  - TP
  - Intune
  - hybrid
---

Android and iOS Enrollment Restrictions explained and demoed

# Intro #

This Blog-post is a bit over-due due to the reason that the highlighted feature was not working when released.
We started testing this feature around the end of June 2017 but it was not fully functional at that time.

After checking every few days it appeared that we were not going to get a quick fix until... today ;-)

Interesting to notice is, that this feature, in the meantime already got added to the latest Current-Branch release (1706) without being functional.

The fact that it "suddenly" started working without any additonal code-change on the Configmgr-product indicates that some changes were made at the backend Intune Service.

So, without further ado, we are going to take a look at another (long) awaited feature , the ability to block users from enrolling personal devices in a hybrid environment.

An obvious pre-requisite here is that you need to have a working intune subscription.

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_Hybrid_Uncompleted.PNG)

# Setting it all up #

Completing the actual challenge (or setup in CB) is not that difficult. Navigate to the Administration pane and select "Microsoft Intune Subscriptions" under the "Cloud Services".

Either Right-click your subscription , select Configure Platforms and choose "Android" (or do the same from the quick action toolbar).

Tick the box that's labeled "Block Personally Owned devices"


![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_BlockAndroidpersonal.PNG)

**Note :** If you have configured your Intune subscription to handle all Android devices as "Android for work", this feature will not work. Microsoft has confirmed that for now, only "non-Android for work" devices are supported in this scenario.

Repeat the process for iOS. in the Popup, select the 2nd tab called "Enrollment Restrictions" and tick that same box to block personally owned devices

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_BlockiOSpersonal.PNG)

That's it ! Congratulations, you've completed another challenge in last month's TP :-)

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_Hybrid_completed.PNG)

Keep in mind that each configuration change can take up to 5 minutes to get processed. DMP-Uploader, the process that is responsible for uploading data to the cloud runs every 5 minutes.

There are exceptions to this process where changes are triggered nearly instant, such as wiping a device.


# The results #

So let's see the effects. I'll first try to enroll an Android device that has not been configured as company owned. Additionaly, I did not enable that all Android devices need to be treated as "Android for work"

If necessary, download the "Company Portal" app from the appropriate store and log in with an account that is allowed to enroll devices into Intune.

Follow the on-screen instructions.

The process is identical with or without this blocking of personal devices and you get the same warnings (eg, that you might need to set a passcode, or have your device encrypted).

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_Enrollment_1.png)
![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_Enrollment_2.png)

When you click the "Enroll" button, you also need to activate the device administrator and at that point the enrollment process starts.

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_Enrollment_3.png)

The device tries to workplace Join and after a few moments you get the notification that your device couldn't be enrolled. The detailed message is :

"Couldn't enroll your device.

Your IT administrator has not authorized this device to enroll. Contact your IT administrator for help."

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_Enrollment_Error.png)

Now, let's register my test-device as a corporate owned device and try the entire process again.

In the Assets and Compliance node, go to the "All Corporate-owned Devices" entry and select "Predeclared Devices"

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_predeclared.PNG)

Right-Click and select "Create Predeclared Devices" (or select it from the quick action toolbar).

Select "Manually add IMEI or serial numbers and details" and select Next.

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_manually_add.PNG)

In the Manual Entry screen select the first line, Provide the IMEI and Operating System of the device you want to enroll and provide details to identify the device if you want.
Click "Next" and finish the wizard.

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_IMEI.PNG)

After refreshing the view in my Admin-UI, we can see that our new device has been added but is not enrolled yet.

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_adminui_predeclared.PNG)

Before proceeding, make sure that your changes have been uploaded to the cloud. Check DMPUploader.Log for entries like shown below.
It shouldn't be a big deal, buf if you eagerly waiting to test out features, 5 minutes could feel like an eternity ;-)

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_DMPUpload.PNG)

After all was synced, I could properly enroll my device.
Checking back the Admin-UI, it now shows my device as being enrolled.

![alt]({{ site.url }}{{ site.baseurl }}/images/TP1706_IMEI_enrolled.PNG)


# Various notes #

*   **At the time of writing not all seems perfect yet. Once you have defined a device as a corporate owned device by predeclaring it, and you later remove the entry from the predeclared devices, you are still able to enroll that device although it's no longer predeclared. Seems that not all settings are replicated properly toward the Intune backend.**

*   Predeclared devices are stored in SQL in the MDMCorpOwnedDevices table (or in WMI in SMS_MDMCorpOwnedDevices).

*   The format for importing through a CSV is : IMEI number (no spaces), IOS Serial number, Operating system, Details.

Only IMEI & Operating system (Windows, IOS or Android) are required. Eg :

111111111111111, ,Windows ,

222222222222222, ,IOS ,

333333333333333, ,Android ,

That's it for now !
