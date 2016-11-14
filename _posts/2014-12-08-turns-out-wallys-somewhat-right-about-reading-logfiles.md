Logging and reading log files has always been a big part of any SCCM admin's life.
It's been a while since I blogged anything, so for those of you wondering yes, I am still alive. Been pretty busy over the past year doing live presentations at several events, but blogging has suffered a bit. This is a first post in an attempt to pickup blogging again, or at least that's the intention.

I'll start of this blogpost with setting something straight, at one of my public speaking sessions a while back, I made the bold statement that Wally was wrong* (about his preferred tool for reading log files). For those of you wondering, it was at a 10 minute presentation I did at the last MMS in Las Vegas as part of the MVP Experts panel. More specifically, it was during session UD-B320: Configuration Manager 2012: MVP Experts Panel. The session itself can still be found online at Channel 9 [http://channel9.msdn.com/Events/Speakers/kim-oppalfens](http://channel9.msdn.com/Events/Speakers/kim-oppalfens "here").

Turns out, as often is the case when you make bold statements, Wally's not wrong at all. In fact he's somewhat right, CMTrace is indeed the second best log viewer tool to read Configuration Manager logfiles.

No, I didn't all of a sudden turn into a notepad believer, the number 1 log file viewer tool for SCCM that I advise everyone to start using is actually part of the new Configuration Manager Support Center. This tool is part of a new toolset in troubleshooting ConfigMgr related issues provided to us by the product team, and can be downloaded here:
[http://www.microsoft.com/en-us/download/details.aspx?id=42645](http://www.microsoft.com/en-us/download/details.aspx?id=42645)

Additionally, you can find an initial description of the tool on the ConfigMgr team blog here: 
[http://blogs.technet.com/b/configmgrteam/archive/2014/05/06/system-center-2012-configuration-manager-support-center-tool-has-been-released.aspx](http://blogs.technet.com/b/configmgrteam/archive/2014/05/06/system-center-2012-configuration-manager-support-center-tool-has-been-released.aspx)

So why do I consider this the best log file viewer available? There are a couple of reasons, but for this blog post I'll just focus on a single use case. Last week, at another happy customer, I had to troubleshoot client to management point communication, more specifically hardware inventory communication. Now, Kim, that's easy enough, anyone with a little operational experience knows that all this takes is looking through the inventoryagent.log on the client, the mp_hinv.log on the management point, and the dataldr.log on the site server. Now, obviously the client I had to troubleshoot was neither running on the mp, nor on the site server, and the site server and management points where obviously on different machines as well. to make matters worse, the customer had no less than 3 MP's to create a nice load balanced environment. Those of you present at my MMS 2013 presentation, or those of you that took the time to actually look at the Channel video will know I am a big advocate for merging logfiles to get the overview of what is going on.

So here's, what I was able to do with the assistance of the new Support Center Logfile viewer:

```posh
.\CMLogViewer.exe ‘\\client\c$\windows\ccm\logs\inventoryagent.log, '\\mpsup1\e$\SMS_CCM\Logs\MP_Hinv.log' , '\\mpsup2\e$\SMS_CCM\Logs\MP_Hinv.log', '\\SiteServer\e$\program files\Microsoft Configuration Manager\Logs\dataldr.log'
```
As you might have noted, all I did was specify the UNC paths to the relevant logfiles and comma separated them. This opens all logfiles I needed merged, without me having to know which mp's the client used to actually send it's hardware inventory along. There's pleny of scenarios where this is useful. Multiple sms providers is another that comes to mind.

PS: There's still a small issue in the Support Center Logfile viewer where it crashes when you specify an inaccessible logfile in this way.

Thank you, [https://blogs.msdn.microsoft.com/ameltzer/](https://blogs.msdn.microsoft.com/ameltzer/ "Adam Meltzer"), for making my life that tad bit easier, yet again.