Hi All,
<h1>Feature Explained</h1>
On the 23rd of September 2015 the ConfigMgr Product team released a blog on an update for Configuration Manager Technical Preview 3, or the new and upcoming version of SCCM, known by many as SCCM vNext. The blogpost contained roughly 321 words. (Yes I did put that in there to weasel you into going in and counting the words )

The shortness of the blog, and it's number of words, in my opinion, doesn't do justice to the importance of this new feature called CM Updates, Updates and servicing or Easy setup, depending on who you talk to. This feature is the way forward for the product team to update the next release of Configuration Manager, and might well replace Service packs and/or CU's in the future. One of the challenges for the CM product team has always been to find the right "release vehicle". Release vehicles are a means to get updates out to the install base, and at present are usually one of these:
<ul>
	<li>New release</li>
	<li>Service pack</li>
	<li>Cumulative Update</li>
	<li>Web release/download</li>
</ul>
The first 2 in that list, are at least perceived as disruptive to the systems management service, while Cumulative Updates have quite different perceptions across different IT departments and companies. But there's definitely a set of IT departments that consider the CU's, and their installation disruptive as well. That means all sorts of things come into play when any of these are released, release management, a rollback plan, the change advisory board to name a few. The ambition of CM updates is to alleviate that pain through an integrated servicing mechanism much like other software that auto-updates, yet with admin approval.

Or in the words of the blog: This update is the first to be delivered through our new "Updates and Servicing" node in the product. Moving forward, this is how you should expect to receive Technical Preview updates. When System Center Configuration Manager becomes generally available in Q4 of this calendar year, we will continue to use this same channel to provide a faster, lighter and easier update experience for new features, bug fixes, and more."

People already involved with Microsoft Intune in a Hybrid scenario will see a lot of similarities with what is called the weave feature to download mobile device management extensions and enable/install these extensions. This should not come as a surprise, as one of the driving factors behind weave was the speed with which new versions and features of IOS, Android and Windows Phone were released. The mobile device management market is rapidly growing and evolving, and needed it's own release vehicle, that release vehicle became 'Extensions for Microsoft Intune' a.k.a weave. Now, with Windows 10 entering the Windows as a service-era, with things like the current branch potentially upgrading every 10 months, Microsoft needed something similar to update general CM items, and not just mobile device management related items. The blog released yesterday was the announcement of "the very first real world public test of this mechanism!". The announcement itself lacked a bit of enthusiasm, but this is a feature that I am excited about for the future of CM.

This feature can update and deliver fixes and features for:
<ul>
	<li>Site servers</li>
	<li>SMS_Provider</li>
	<li>Configuration Manager console</li>
	<li>Configuration manager clients</li>
</ul>
Which should provide the necessary flexibility to update just about anything, as this allows the product team to update, the ConfigMgr File system, the Admin UI file system, the database (Data as well as new tables, views, Stored procedres,functions, as WMI)

<h1>Feature documentation</h1>
The announcement itself didn't go into much detail explaining how the future worked, however did contain a <a href="http://blogs.technet.com/b/mniehaus/archive/2012/09/02/speed-up-mdt-task-sequences-in-configuration-manager.aspxhttps:/technet.microsoft.com/library/dn965439.aspx">link</a> to the documentation of the Microsoft System Center Configuration Manager Technical Preview. This page contains a section on "Updates and servicing" which gives some insight into how this all operates.

Some important items in that documentation, are highlighted below, but I do encourage you to go and read the doc itself to get all the nitty gritty details

First of all, this entire process is driven by a new site system role called the cloud connection point, which replaces the Microsoft Intune connector site system role. In the current tech preview (Release 3), <span style="color: red;">the cloud connection point needs to be installed on the primary site server and needs internet access. </span>Both of these limitations are expected to be resolved in a subsequent release, and an offline procedure is planned for a future release, as mentioned in the comments section of the blog.

Other limitations: Os language must be English, and you must set your date/time format to this absolutely weird way of throwing days, months and years in some random order called MM-DD-YYYY. Not sure who ever came up with that way of noting dates, but apparently that's what we need to use. In testing and hearing experiences from others, <span style="color: red;"><strong>this is the number one deal-breaker: The update process itself seems to be rock-solid, assuming you set your Date/time format accordingly.
</strong></span>

New updates are checked for every 7 days, since the install date of the environment. According to the docs a restart of the SMS_executive service triggers the check for updates as well. I'd venture a guess that restarting the SMS_DMP_Downloader might trick it as well, which would be less disruptive. (To be tested).

Once the updates are downloaded you'll find them in <strong>Administration</strong> &gt; <strong>Cloud Services</strong> &gt; <strong>Updates and Services</strong> as available, you can subsequently click <strong>Install Update Pack
</strong>

As a final note, just running the prerequisite checker standalone, from this same node, doesn't work in the current build. Triggering this will perform the install as well.
<h1>Feature at work – The server upgrade</h1>
<ol>
	<li>Step 1 would be to restart your sms_executive service if the update hasn't arrive in your <strong>Updates and Services</strong> node.</li>
	<li>The update should arrive and be in the downloading state for a while. You can monitor the download progress in the DMPDownloader.log the log should contains lines similar to:</li>
</ol>
<em>EasySetupDownload thread is starting... $$&lt;SMS_DMP_DOWNLOADER&gt;&lt;09-22-2015 21:21:36.981-120&gt;&lt;thread=4700 (0x125C)&gt;
</em>
<p style="margin-left: 18pt;"><em>Download Easy setup payloads~~ $$&lt;SMS_DMP_DOWNLOADER&gt;&lt;09-22-2015 21:21:37.008-120&gt;&lt;thread=4700 (0x125C)&gt;
</em></p>
<p style="margin-left: 18pt;"><em>Get manifest.cab url~~ $$&lt;SMS_DMP_DOWNLOADER&gt;&lt;09-22-2015 21:21:37.012-120&gt;&lt;thread=4700 (0x125C)&gt;
</em></p>
<p style="margin-left: 18pt;"><em>Successfully write the update meta into outbox for package dcd17922-2c96-4bd7-b72d-e9159582cdf2~~ $$&lt;SMS_DMP_DOWNLOADER&gt;&lt;09-22-2015 21:40:36.934-120&gt;&lt;thread=4700 (0x125C)&gt;
</em></p>


