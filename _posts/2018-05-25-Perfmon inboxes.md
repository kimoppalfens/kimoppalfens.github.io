---
title: "Getting alerts on Inbox backlogs"
header:
author: Tom Degreef
date: 2007-05-25
categories:
  - SCCM
  - Configmgr
  - Inboxes
  - Perfmon

tags:
  - SCCM
  - Configmgr
  - Inboxes
  - Perfmon
---

Receiving alerts on Inbox backlogs

# Intro #

During our [MMSMOA session on Inboxes A-Z](https://mms2018.sched.com/event/EeQx/configmgr-inboxes-a-through-z), I quickly demonstrated how you can use performance monitor (perfmon) to trigger a scheduled task when an inbox goes over a specified threshold.  
Kim was kind enough to announce that I would blog the entire setup, so here we are :-)

# Background #

As we explained during our presentation, most Inboxes should be (nearly) empty or at least clearing up backlog constantly, but the seasoned ConfigMGr admin knows that this is not always the case and that from time to time not all files are processed. Depending on the inbox this is happening on, it could cause issues in your environment like delayed deployment statuses , clients not receiving policy, no up to date hardware inventory, ...

There is the inbox monitor log (inboxmon.log) but apart from showing you the file-count in the "monitored" inboxes, it doesn't generate any alert at all. Besides that, it probably doesn't monitor all the inbox that you are interested in, so it sounds a bit useless. However the inbox monitor is responsible for creating the performance counters that we are going to rely on for receiving alerts. Let's dig deeper !

# Inbox Monitor #

So, the first part of the puzzle is the inbox monitor, it "counts" the number of files on a 15 minute (900 seconds) cycle and reports those numbers in the inboxmon.log  
**note :** The 15 minute interval is defined in the "site control file". We updated the frequency in our lab for testing purposes but in a regular environment, the default value should be good enough.

Now, if you take a look at the inboxmon.log, you'll notice that it doesn't necessarily monitor all the inboxes you could be interested in. There's an easy fix for that. On your site-server, open up the registry and navigate to HKLM\Software\Microsoft\SMS\Inbox Sources\Inbox Instances  
You'll notice a list of keys ranging from 0 to 80, representing most "important" inboxes. I'm going to focus on the policy provider in this blog as an example. A backlog of files in there can have some serious impact on your clients (not) receiving their policies, but... it's not monitored by default.

In the registry, if we navigate to the subkey 71 (HKLM\Software\Microsoft\SMS\Inbox Sources\Inbox Instances\71), you'll notice a Dword value "Monitoring Enabled" that is set to 0

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes1.PNG)

If we flip that value to 1, our inbox becomes monitored. Once the 15 minute monitoring cycle is triggered again, you should see the file count for policypv in the log. In addition, you'll also notice that a new performance monitor is created. That's another part of our puzzle.

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes2.PNG)

**note :** The easiest way to locate the correct subkey is simply to start a search at the "Inbox Instances" key for the name of the inbox you want to monitor, in my case that was "Policypv.Box". Do notice that some subfolders of inbox can have their own registry key. Eg, Policytargeteval is a subfolder of policypv.Box and has key 70 assigned to it, so double-check that you monitor the correct key.

# Scheduled Tasks #

Yes... good old scheduled tasks :-)
Out inbox monitoring solution relies on scheduled tasks. Fire up Task Scheduler and create a new task.  
Give it a meaningful name (you'll need it later), set it to run whether user is logged on or not and make sure it is allowed to run with the highest privileges

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes3.PNG)

No need to define any triggers as we will use perfmon for that.  
On the Actions Tab, define a new action that will be executed when the task is triggered.  
I opted to run a simple PowerShell script that send me a notification email([Source](https://gallery.technet.microsoft.com/scriptcenter/Simple-Powershell-function-8e826d7c)) , but the possibilities are only limited to your imagination (and scripting skills maybe)

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes4.PNG)

As you can see, I trigger the program "C:\Windows\System32\WindowsPowerShell\v1.0\Powershell.exe" with an argument of "-File "C:\Script\SendMail.PS1"

With that in place, we can move over the the last part of solution

# Performance Monitor #

Open Performance monitor and browse down to "Data Collector Sets \ User Defined". Right-click on "User Defined" and select "New \ Data collector Set"

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes5.PNG)

Again, provide a meaningful name and "create manually (Advanced)". Click Next

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes6.PNG)

Select "Performance Counter Alert", Click Next

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes7.PNG)

Click the "Add..." button to add new performance counters.  
Browse down to "SMS Inbox" and select the counter(s) you want to use. In my case, Policypv.box.  
Select "All instances" and click "Add >>".  
If you are satisfied with your selection. Click OK.

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes8.PNG)

Select the alerting limit and select "Finish"

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes9.PNG)

It's hard to provide guidance on what limit should be used as it is really something that is environment specific. It could very well be that files in the inboxes are processed at different speeds, depending on the day or time or what the site-server is currently processing at that moment in time. You can always go back and tweak the limits if alerts come in to fast (or to slow) at any given time.

Once you clicked Finish, select your newly created data collector set in the left-hand pane so that the actual data collector becomes visible

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes10.PNG)

Take the properties of your data collector (in my case called DataCollector01) and update the sample interval. Make sure to match it to the interval cycle of your inbox monitor (15 minutes if you haven't changed it).  
The values of the performance counters are only updated each time the inbox monitor starts a new cycle. If your sample interval is faster, you will get multiple alerts for the same "issue" and that doesn't make sense.

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes11.PNG)

On the "Alert Task"-Tab, enter the name of the scheduled task you created in the previous step (Copy/Paste to avoid typo's) and Click OK.  
**note :** You can provide arguments such as counter that triggered the alert. They could be useful to create a more tailored email.

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes12.PNG)

Last, but not least, also take the properties of your data collector set itself.

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes13.PNG)

check the "Stop Condition"-tab to make sure that your data collector doesn't automatically turn itself off. You most likely want continuous monitoring of your inboxes.  
Finally, on the "Task"-tab you can add the name of another scheduled task that will run when your data collector set is stopped (for whatever reason). This would allow for some basic monitoring of the performance monitor itself.

All that is left to do, is to start the actual performance monitor and test it.

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes14.PNG)

I created a few text-files in my policypv.box, waited for the cycle to run and got my notification email...

![alt]({{ site.url }}{{ site.baseurl }}/images/Inboxes15.PNG)

**note :** If your scheduled task doesn't seem to get triggered, it could very well be that your performance monitor hasn't started it's sample interval. If you set it to 15 minutes as I suggested, it does not necessarily run immediately after your inbox monitor's schedule, but it could be anywhere between 1 second and 15 minutes after that, depending on when you started the monitor.

# Conclusion #

With these simple steps it becomes a bit easier to keep an eye on the critical inbox component of ConfigMgr. It is up to you to start benchmarking your environment and enable monitoring on those inboxes that you had trouble with in the past. You'll notice that some of those inboxes are not monitored by default at all, but the fix is easy :-)

Enjoy and let me know what you managed to cook up! 