---
title: "Defending the (SCCM) Castle"
header:
author: Tom Degreef
date: 2019-12-12
categories:
  - SCCM

tags:
  - SCCM
  - Configuration Manager
  - Security
---

With great power comes great responsibility !

## Intro ##

The following blog post is a summary of the lessons learned and offered, worldwide, in our [SCCM Vulnerability assessment offer](http://www.oscc.be/osccservices/Security-Audit/). If this is something that sounds of interest to you, and it should, don't hesitate to contact us.

If you need additional feedback on this offer, here is some of the feedback of speaking sessions we've done that mention a subset of what the Vulnerability assessment contains:
* **"Very interesting topics and demos.  All SCCM admins should attend sessions like this."**
* **"Best session of MMS...and no i didnt say that on all the evals. More security deep dives like this please!"**
* **"This session should be mandatory for all attendees. Great insights on how to handle security with config manager. Speakers were very passionate about their topic"**
* **"best session I've seen all week! not only entertaining, understandable examples, but simple mitigations that actually matter are very helpful. Thanks!"**
* **"Loved it and would love to have had the chance to send more of my staff to it. "**

## Setting the stage ##

Every action taken by ConfigMgr is executed with the highest privileges.

If your ConfigMgr environment gets compromised and executes malicious code, would you notice ?
The extra (network) traffic it would generate will probably not stand out as it would look like regular ConfigMgr traffic.

Even a [Windows Defender Application Control](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/windows-defender-application-control) implementation would probably not protect you, as chances are that you listed Configmgr as a "managed installer", trusting everything it executes.

**note:** The above statement should not be used as an excuse not to implement application whitelisting! We still strongly believe that application whitelisting is one of the [best defences against malware](http://www.oscc.be/osccservices/Windows-Defender-Application-Control/).

The point really is that you should not take security of your ConfigMgr environment lightly, because it might be very difficult to detect abuse.

We have already spent 2 ([1](https://sched.co/N6f0),[2](https://sched.co/N6f3)) full sessions at MMSMOA 2019 on this topic and a "demo-only" session at MMS Jazz Edition. In Belgium we presented this at Lowlands Unite 2019.

This blog post is meant as a summary for what we have shown before.

## SQL Security ##

Did you know that any local administrator of a server that hosts a SQL instance can become a SQL admin of that instance ? This process is actually [documented by Microsoft](https://blogs.technet.microsoft.com/sqlman/2011/06/14/tips-tricks-you-have-lost-access-to-sql-server-now-what/). I must admit, this was a surprise to me the first time I learned about this "feature"

And guess what... Once you have obtained SQL admin right , you can become a ConfigMgr Admin in no time too. To achieve that, you swap the SID from an account with admin rights in ConfigMgr, with your own SID. At that point, you can open the ConfigMgr console with your own account, however, the Admin-UI will still list the username of the person you hijacked the SID from. (And the hijacked person no longer has access to configMgr)

We often come across environments where (some) users were granted permissions directly and through an AD security group. In that scenario, if someone hijacks the SID of one of those users, they suddenly both have access (he/she still would have access throught the AD group) and that would be very difficult to notice.

So, Local Admin = SQL Admin = Configmgr Admin. Make sure that you trust those local admins on the SQL box that hosts the Configmgr database ;)

There isn't really a defense against this attack. Make sure to tightly control the local admins on your SQL server and to manage access to Configmgr only through either AD groups or direct membership, not both. Oh...and clean up those accounts of people who left the company a while ago :)

You also might want to run the SQL query below. This should give no results...if it does, it means a discrepancy has been found between the SID of an admin and his actual AD SID.

```sql
select unique_user_name0,sid0, SID_binary(sid0),adminsid,logonname from v_r_user usr
join rbac_admins rbac on rbac.logonname = usr.unique_user_name0
where SID_binary(sid0) <> adminsid
```

## Status Filter Rules ##

Status filter rules are so incredibly powerful ! A little bit to powerful at times ?

During our sessions at MMSMOA 2019 we wanted to show a flaw with Status Filter Rules, however we were kindly requested by Microsoft to not reveal this flaw until they managed to fix it.
The fix was introduced in ConfigMgr release 1906 (So upgrade your environments ASAP!)
This serves as a great reminder, even if you don't consider the new features in a build breathtaking, you should still upgrade. As with all other software that gets updated, buts and security issues are solved with each release. From a security perspective leaving your systems management platform unpatched is a really bad decission.

This flaw deserves a post of it's own ! Stay tuned.

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending05.png)


## Attacking Client Push ##

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending02.jpeg)

If there is one client installation method that sticks out like a sore thumb it must be Client Push!
Not only does it require a ton of prerequisites before it actually works, but it can be abused in a man in the middle attack.

**Note:** One of those prerequisites is that the account used for client push has local admin privileges on your workstations

[Every admin should at least understand the basics of the man in the middle attack, however it seems a lot of people don't.](https://mez0.cc/posts/ntlm-relaying.html) To make it as comprehensible as possible, we did a little role-playing at MMS JE to explain a MITM attack with the great help of Arne Smeyers [@arnenysd](https://twitter.com/ArneNYSD), one of our audience volunteers. The following explanation isn't 100% technically accurate, the sketch is simplified to make grasping the mechanism of an NTLM/SMB relay MITM attack easier.

Regular NTLM authentication is as follows :

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending03.png)

* Computer A requests access to Computer B
* In order to grant access, Computer A must perform a calculation of a random challenge (presented by Computer B) with the hash of his password
* Computer B performs the same calculation and if the results match, access is granted

A man in the middle attack could be explained like this :

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending04.png)

* User A requests access to Computer B
* Our MITM machine C is in between this communication and intercepts this request
  * When receiving this request, the MITM machine requests access to Computer B
  * Computer B sends a challenge to the MITM machine
* the MITM machine forwards the challenge to User A
* User A calculates the result of the challenge by hashing it with the hash of his password and sends this to the MITM machine
  * The MITM machine forwards the results of those calculations to Computer B
  * Computer B performs the same calculations, sees that it matches and grants access to the MITM machine

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending06.png)

Now, how does this relate to Client Push ? Well, for this to work, I have to convince a user, preferably a highly privileged one to connect to my machine.

If an attacker could trick you into performing a client push to a machine he controls, he could use that machine as her MITM machine. Keep in mind that the client push account typically has local admin rights on all clients... As such she could redirect your client push to a device that she need local admin rights on.

The good news is, there is an easy fix for this. This MITM attack relies on NTLM. If you still insist on keeping client push around, make sure that it uses only Kerberos authentication and that NTLM fallback is disabled.

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending07.png)

## Decyphering the Network Access Account ##
Next demo had us, representing some bad actors, decypher your Network Access Account. Re-enforcing the message that you should consider the NAA as an account to which everybody in the company, or with access to a single machine in your company has access.

In this demo we were assisted by Dawn "[ConfigGirl](https://configgirl.com/author/dawnwertz/)" Wertz [@wertzdm3](https://twitter.com/wertzdm3). She picked a child-safe password for us to decrypt. We promise that this password is as close a call we'll ever have to having cat pictures in our presentations.

Roger Zander was the first to publicly state this over 4 years ago in his blog [Network Access Accounts are Evil](http://rzander.azurewebsites.net/network-access-accounts-are-evil/).

Kim hasn't been a huge fan of the fix Roger suggested at the time though. His proposal is to make sure the account is as underprivileged as it can be. Meaning this should be a regular domain user with some additional limitations enforced as documented [here](https://docs.microsoft.com/en-us/configmgr/core/plan-design/hierarchy/accounts#network-access-account).

What the documentation doesn't spell out is that you can "grant" this account all the other deny user rights on a distribution point, and all the deny user rights on all machines that aren't acting as distribution points. To assist in making this configuration OSCC has [blogged](http://www.oscc.be/sccm/configmgr/Protecting-the-Network-Access-Account-using-Configuration-Items-Quest-Part1/) a CI in the past that doest exactly this for you.

Now, Configuration Manager Current Branch evolves at speed. One of its more recent evolutions introduced [Enhanced HTTP](https://docs.microsoft.com/en-us/configmgr/core/plan-design/hierarchy/enhanced-http). One of the goals at introduction of this feature was to eliminate the need for the Network Access Account all-together. We, and the product team, believe this has been achieved in the more recent builds of Configuration Manager.
We strongly suggest putting this on your todo list to test and remove the Network Access Account from your environment.

## Hiding applications in your Admin UI ##

Aaaah, this one is one of my favorites. Playing around with PowerSCCM (although it stricktly isn't needed).
A while ago, at one of the DefCon conferences there was a session around Configmgr. From a configmgr point of view, it wasn't that spectacular, but it did show that the infosec community is showing interest in the product we so love!

The infosec community did produce a powershell module called [PowerSCCM](https://github.com/PowerShellMafia/PowerSCCM). It is a mix of functions that allow you to gather information about an SCCM environment. Next to that, it allows you to create collections, applications, deployments...

Nothing spectacular so far, right ? But it does have a few neat tricks up its sleeve.

It can create hidden applications. These are just regular applications that are not visible within the Configmgr Admin UI. Under the hoods the module just sets a WMI property for that specific application.

If you are suddenly worried about your own environment, you could run the following SQL Query. It will show you how many hidden applications you have :

```sql
select * from ci_configurationitems where ishidden = 1 and citype_id = 10
```

If you did find some hidden applications, we'd be very interested in learning about this, as your environment is probably compromised.

PowerSCCM can also deploy applications/payload without that content being on a DP. What this means is, it can hide payload in WMI. It can convert payload to Base64 encoding and just include it in the command line itself! That's probably something most of us would never think of, but it makes one hell of a mix ! It makes the application execution file-less. Being able to execute things without writing to disk is a huge boon for InfoSec red-teamers. Adding my secret payload to a hidden application that gets executed on every device with local system privileges without writing to disk. Sounds like the red teamers holy grail. Good luck blue team detecting this at a client.

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending08.jpg)

Finding applications without content is left as an exercise to the reader.

# Conclusion #

A systems management tool is a powerfull tool for good, with this great power comes a certain level of risk though. As a systems management admin security best practices and knowledge should be top of mind. The system you manage is one avenue to full company compromise. We've started to give ConfigMgr based training, speaking sessions and security reviews of Microsoft's systems management products and cloud services a while ago and will continue our path in 2020.

Should you want to learn more about our SCCM Vulnerability assessment after reading this post, you can find more info [here](http://www.oscc.be/osccservices/Security-Audit/). Alternatively, just [contact us](http://www.oscc.be/Contactus/) and shoot any questions you have our way.