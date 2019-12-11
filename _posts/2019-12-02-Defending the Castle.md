---
title: "Defending the (SCCM) Castle"
header:
author: Tom Degreef
date: 2010-12-02 
categories:
  - SCCM

tags:
  - SCCM
  - Configuration Manager
  - Security
---

With great power comes great responsibility !

# Intro #

To start of with the words of a wise man :

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending01.jpg)

Who am I to question spiderman? 
The fact is that our beloved Configmgr, in most environments, manages all devices. Both servers and workstations. 
Every action taken by Configmgr is executed with the highest privileges.

If I somehow manage to compromise your Configmgr environment and make it execute some malicious code, would you notice ?
Extra (network) traffic it would generate will probably not stand out as it would look like regular Configmgr traffic.

Even [Windows Defender Application Control](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/windows-defender-application-control) will probably not even protect you at this point because chances are that you listed Configmgr as a "managed installed", as such trusting everything it executes.
**note:** The above statement should not be used as an excuse not to implement application whitelisting! We still strongly believe that application whitelisting one of the [best defences against malware](http://www.oscc.be/osccservices/Windows-Defender-Application-Control/).

The point really is that you should not take security on your Configmgr environment lightly, because it might be very difficult to detect abuse.

We have already spent 2 ([1](https://sched.co/N6f0),[2](https://sched.co/N6f3)) full sessions at MMSMOA 2019 on this topic and a "demo-only" session at MMS Jazz Edition. In Belgium we presented this at Lowlands Unite 2019.

This blog post is meant as a summary for what we have shown before.

## SQL Security ##

Did you know that any local administrator of a server that hosts a SQL instance can become a SQL admin of that instance ? This process is actually [documented by Microsoft](https://blogs.technet.microsoft.com/sqlman/2011/06/14/tips-tricks-you-have-lost-access-to-sql-server-now-what/). I must admit, this was a surprise to me the first time I learned about this "feature"

And guess what... Once you are SQL admin, you can become Configmgr Admin. What happens in the background is that we swap the SID from an account with admin rights in Configmgr, with our own SID. At that point, you can open the Configmgr console with your own account, however, the Admin-UI will still list the username of the person you hijacked the SID from. (And the hijacked person no longer has access to configmgr)

We often come across environments where (some) users were granted permissions directly and through an AD security group. In that scenario, if I hijack the SID of one of those users, we suddenly both have access (he/she still would have access throught the AD group) and that would be very difficult to notice

So, Local Admin = SQL Admin = Configmgr Admin. Make sure that you trust those local admins on the SQL box that hosts the Configmgr database ;)

There isn't really a defense against this attack. Make sure to tightly control the local admins on your SQL server and to manage access to Configmgr only through either AD groups or direct membership, not both. Oh...and clean up those accounts of people who left the company a while ago :)

You also might want to run the SQL query below. This should give no results...if it does, it means a discrepancy has been found between the SID of an admin and his actual AD SID.

```
select unique_user_name0,sid0, SID_binary(sid0),adminsid,logonname from v_r_user usr
join rbac_admins rbac on rbac.logonname = usr.unique_user_name0
where SID_binary(sid0) <> adminsid
```

## Status Filter Rules ##

Status filter rules are so incredibly powerful ! Maybe they are a little bit to powerful ?

During our sessions at MMSMOA 2019 we wanted to show a flaw with Status Filter Rules, however we were kindly requested by Microsoft to not reveal this flaw until they managed to fix it. 
The fix was introduced in Configmr release 1906 (So upgrade if you use Status filter rules!) However this flaw deserves a post of it's own ! Stay tuned.

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending05.png)

Now it is also worth nothing that if you use Status filter rules to run scripts, that you need to be careful who has access to these scripts. 
Make sure that your properly shield them off with NTFS permissions so no one could alter those scripts. Imagine if I could just add some lines of code to your scrip, but leave the rest as is...
You would probably never find out and each time the SFR fires, it would also execute my code.

## Attacking Client Push ##

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending02.jpeg)

If there is one client installation method that sticks out like a sore thumb it must be Client Push! 
Not only it requires a ton of prerequisites before it actually works but it could very well be used for a man in the middle attack.

**Note:** One of those prerequisites is that the account used for client push has local admin privileges on your workstations

Everybody knows about the man in the middle attack, however it seems most people don't really know how this is actually works. To demonstrate this, we did a little role-playing at MMS JE to explain a MITM attack. The following explanation isn't 100% accurate but it is sufficient enough to grasp the idea of a MITM attack.

Regular NTLM authentication is as follows :

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending03.png)

* Computer A requests access to Computer B
* In order to grant access, Computer A must perform a calculation of a random challenge (presented by Computer B) with the hash of his password
* Computer B performs the same calculation and if the results match, access is granted

A man in the middle attack could be explained like this :

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending04.png)

* Computer A requests access to Computer B
* Our MITM machine is in between this communication and intercepts this request
  * in it's turn, the MITM machine requests access to Computer B
  * Computer B sends the challenge to the MITM machine
* the MITM machine forwards the challenge to Computer A
* Computer A calculates the result with the hash of his passwords and sends this to the MITM machine
  * The MITM machine fowards the results of those calculations to Computer B
  * Computer B performs the same calculations, sees that it matches and grants access to the MITM machine

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending06.png)

Now how does this relate to Client Push ? What if I could trick you into performing a client push to my machine, except my machine will be the MITM machine under my control and I target another machine. Keep in mind that client push has local admin rights... As such I could redirect your client push to a device that I need local admin rights on.

Good for both of us, there is an easy fix for this. This MITM attack relies on NTLM. If you still insist on keeping client push around, make sure that it uses only Kerberos authentication and that NTLM fallback is disabled.

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending07.png)

## Decyphering the Network Access Account ##


## Hiding applications in your Admin UI ##

Aaaah, this one is one of my favorites. Playing around with PowerSCCM (although it stricktly isn't needed).
A while ago, at one of the DefCon conferences there was a session around Configmgr. From a configmgr point of view, it wasn't that spectacular, but it did show that the hacking community is showing intrest in the product we so love! Now if we would combine forces, our knowledge of Configmgr with their knowledge of hacking... that would become a pretty powerfull mix. Good thing we are still on the good side :)

But that hacking community did produce a powershell module called [PowerSCCM](https://github.com/PowerShellMafia/PowerSCCM). It is a mix of functions that allow you to gather information about an SCCM environment. Next to that it allows you to create collections, applications, deployments...

Nothing spectacular so far, right ? But it does have a few neat tricks up its sleeve :

It can create hidden applications : It's just a regular application that is not visible within the Configmgr Admin UI. Under the hoods they just set a WMI property for that specific application, but it is something at least I didn't expect to be possible. 

If you are suddenly worried about your own environment, you could run the following SQL Query. It will show you how many hidden applications you have :

```
select * from ci_configurationitems where ishidden = 1 and citype_id = 10
```

If you did find some hidden applications, we wouldn't mind you telling us :)

But PowerSCCM can also deploy applications/payload without that content being on a DP. What this means is, it can hide payload in WMI (the M stands for 'Magic'), but it could equally convert any payload to Base64 encoding and just include it in the command line itself! That's probably something most of us would never think of, but it makes one hell of a mix !

Adding my secret payload to a hidden application that gets executed on every device with local system privileges. Sounds like hacker's heaven to me.

![alt]({{ site.url }}{{ site.baseurl }}/images/Defending/Defending08.jpg)

Unfortunately I don't have an off-the-shelve query to see if this has happened in your environment.

# Conclusion #

