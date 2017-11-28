---
title: "An interesting use-case for Intune and SCCM Co-Management - Part 4"
header:
author: Tom Degreef
date: 2007-11-27
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

[Part 1 - Cloud management Gateway](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/CoMGMT-usecase-Part-1/)
[Part 2 - AAD Discovery](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/azure%20ad/aad/CoMGMT-usecase-Part-2/)
[Part 3 - Co management](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/azure%20ad/aad/CoMGMT-usecase-Part-3/)

So, we finally have all the pre-requisites in place and are ready to pilot a workload. 
As you could see in Part 3, we have configured "Compliance policies" as our first (and only) workload to pilot.

There is actually a specific reason for choosing Compliance Policies. 
If you have enabled the pre-release feature called "Conditional access for managed PC's" , you can actually create Compliance policies for Configuration Manager managed Pc's. 
However, if you compare the list of available policies to the list of policies that we have in Intune, there is a rather large gap.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl1.PNG)

The list above are the 5 rules available for Configmgr managed pc's.

If we compare that to what's available in Intune :

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl2.PNG)
![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl3.PNG)
![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl4.PNG)

As you can judge for yourself, a rather large gap ! 

There is however 1 "work-around" here and that is going to the "Hybrid" scenario and having your windows 10 pc's managed "only" through Intune. Then, the same set of policies is available from within the ConfigMgr Admin UI.  
This is only available in "Hybrid-Mode", thus you have to decide if you want your Windows 10 pc's managed by **Intune OR ConfigMrg**  

In the scenario that we have been working on the past few blogposts, we can have a PC managed by **Intune AND ConfigMgr**, making the best of both worlds available to us.  
In this particular case, it is rather obvious that Intune excels in the amount of compliance policies available, so it makes a lot of sense to have Intune manage that part and use ConfigMgr for the other workloads.

We will be using the Azure AD Application proxy as the resource we want to access using Conditional Access. Setting up the Azure AD Application proxy is not part of this blogpost, but I used partially [this](https://www.petervanderwoude.nl/post/conditional-access-for-published-configmgr-reports/) blog to guide me through. 

# Preparing for Conditional Access #

Before we move on, make sure that the Windows 10 machine you were testing with (and that should be now in a Co-Managed state) is part of that Pilot collection we used in Part 3. That way, we make sure that compliance policies are managed through Intune for that machine.

## Setting up Compliance Policies ##

Logon to your Azure Portal and navigate to Intune. 
Select "Device Compliance"

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl5.PNG)

In the Device Compliance node, select "Policies" and create a new Policy

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl6.PNG)

Provide a meaningful name and description and select "Windows 10 and Later" as the platform.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl7.PNG)

On the "Settings" part, click "Configure".  
The 3 subparts available here contain the set of compliance rules I posted in the Intro part.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl8.PNG)

Create a compliance policy to fits what you need in your environment. For my blog I'll go with the following :

- Device Health
    - Require Bitlocker
    - Require SecureBoot
- Device Properties
    - Minimum OS version (10.0.16299)
- System Security
    - N/A

**Note :** for OS version checks, make sure that you use whatever "Winver" returns as an OS Version. For Windows 10, this would be, "10.0." + the buildnumber, so for Windows 10 - 1709 this translates into : 10.0.16299

The result looks like this 

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl9.PNG)

Click "Create" to actually save your policy. Once you do that you end up in a new window.  
On the Properties menu, you can adjust the rules we just set, but at this point we are more interested in assigning this Policy.  
Click the 'Assignment button.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl10.PNG)

If you have pre-created user-groups, you can assign them here, but in my test-environment I'll assign this policy to 'All Users'.  
Make sure to Save your assignment.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl11.PNG)

Important to notice here is that if these rules are validated by the client, the machine ends up in either the "Compliant" state or the "Not Compliant" state. This is important for our next step !

## Setting up Conditional Access ##

Conditional access, as the name implies, allows you to access a certain resource if you meet all of the required conditions. On of those conditions can be that you need to be marked as "Compliant" (see previous steps).  
It basically prevents access to your company resources if you do not meet a set of required conditions and it is a bit of a paradigm shift in how IT works.  
We used to have this method of trying to protect the device as much as we can (anti-virus, anti-theft, ...) but with conditional access we shift that away from the device and to the data.  
This makes a lot more sense as users are now able to access corporate data from a lot more devices than in a "traditional" world and we might not have full control over all those devices.  
With conditional access in play, we don't need to have full control. We just lay out some ground rules, and if those are met, then we trust the device secure enough to access the corporate data. If not, users won't have access to their mail, files, ... or whatever you want to protect.

Let's enable this fantastic feature !

Back in the Intune main-page, select "Conditional Access"

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl12.PNG)

In the new windows, create a new policy

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl13.PNG)

Provide again a meaningful name for the policy you are creating (you can create different policies for different resources). Select the users you want to assign it to (it makes sense to at least target the same users you target with the compliance policy) and select the cloud apps you want to protect. 
In my case, that would be the Configmgr Reporting website I have published through the Azure AD application proxy.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl14.PNG)

For the conditions, you can specify additional requirements that you want to test for such as 

- Sign in Risk
- Device Platforms
- Locations
- Client Apps

If you feel this is needed in your environment, feel free to enable any of these settings, but for this blog we will focus on being "Compliant" with our compliance policy we created before.

Under the Access controls menu, Select "Grant".  
Grant access once the device is marked as Compliant.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl15.PNG)

Finally, Enable the policy and Create it.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl16.PNG)

That should be all there is to it. Let's verify if indeed this has the wanted effect on our test-client.

# The results #

On my specific test-machine, I have removed Bitlocker, so it should be marked as "non compliant".

