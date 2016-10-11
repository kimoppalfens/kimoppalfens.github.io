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

