---
title: "An interesting use-case for Intune and SCCM Co-Management - Part 5"
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

Real-World scenario on where Intune and SCCM Co-management could come in handy.  Configuring Conditional Access.

# Intro #

[Part 1 - Cloud management Gateway](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/CoMGMT-usecase-Part-1/)
[Part 2 - AAD Discovery](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/azure%20ad/aad/CoMGMT-usecase-Part-2/)
[Part 3 - Co management](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/azure%20ad/aad/CoMGMT-usecase-Part-3/)
[Part 4 - Deploying the ConfigMgr Agent through Intune](http://www.oscc.be/sccm/configmgr/intune/co-management/cloud%20management%20gateway/cmg/azure%20ad/aad/CoMGMT-usecase-Part-4/)

Let's bring these Co-Management blog series to an end and tie it all together with conditional access.
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

We will be using Configmgr reporting website as the resource we want to access using Conditional Access.  
In my scenario, my Reporting point is co-located on my primary site and as such only available on-premise. However, the Azure AD Application proxy can be used to make on-premise resources available through "the cloud", and as an additional benefit, we can apply some conditional access rules on it.
Setting up the Azure AD Application proxy is not part of this blogpost, but I used partially [this](https://www.petervanderwoude.nl/post/conditional-access-for-published-configmgr-reports/) blog to guide me through. 

# Preparing for Conditional Access #

Before we move on, make sure that the Windows 10 machine you were testing with (and that should be in a Co-Managed state) is part of that Pilot collection we used in Part 3. That way, we make sure that compliance policies are managed through Intune for that machine.

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
    - N/A
- Device Properties
    - Minimum OS version (10.0.16299)
- System Security
    - Encryption of data storage on device (Require)

**Note :** for OS version checks, make sure that you use whatever "Winver" returns as an OS Version. For Windows 10, this would be, "10.0." + the buildnumber, so for Windows 10 - 1709 this translates into : 10.0.16299.

The result looks like this 

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl9.PNG)

Click "Create" to actually save your policy. Once you have done that, you end up in a new window.  
On the Properties menu, you can adjust the rules we just set, but at this point we are more interested in assigning this Policy.  
Click the 'Assignment' button.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl10.PNG)

If you have pre-created user-groups, you can assign them here, but in my test-environment I'll assign this policy to 'All Users'.  
Make sure to Save your assignment.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl11.PNG)

Important to notice here is that if these rules are validated by the client, the machine ends up in either the "Compliant" state or the "Not Compliant" state. This is important as we go forward !

## Validating our Compliancy ##

Let's for a moment check on our client what the effects are of these new compliance rules. 
Logon to your test machine and force an Intune policy sync to make sure that our compliance policy is applied to this client. (see part 4 on how to do that).  
Given that we are "Co-Managed", we can actually verify our compliancy state from the client itself. Open up your software center and click the "Device Compliance" tab.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl17.PNG)

As we can see, we are not compliant because we are lacking disk encryption.  I'm not going to remediate it at this point yet as we want to validate conditional access first.

On the Intune portal, we can equally see that our test-device isn't compliant.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl18.PNG)

# Setting up Conditional Access #

Conditional access, as the name implies, allows you to access a certain resource if you meet all of the required conditions. One of those conditions can be that you need to be marked as "Compliant" (see previous steps).  
It basically prevents access to your company resources if you do not meet a set of required conditions and that is a bit of a paradigm shift in how IT works.  
We used to have this method of trying to protect the device as much as we can (anti-virus, anti-theft, ...) but with conditional access we shift that away from the device and to the data.  
This makes a lot more sense as users are now able to access corporate data from a lot more devices than in a "traditional" world and we might not have full control over all those devices anymore.  
With conditional access in play, we don't need to have full control. We just lay out some ground rules, and if those are met, we trust the device secure enough to access the corporate data. If not, users won't have access to their mail, files, ... or whatever you want to protect.

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

**Note :** Just like with Configmgr not being a real-time product, it seems that the cloud-world isn't all that different. Every policy you create in the portal takes time to apply. In a real-world scenario, this isn't a huge deal but when you are validating things in a small lab, take into account that there is some delay between you enabling/enforcing a policy and your client being "aware" of it.

# The results #

We verified before that our test-machine isn't compliant, so the result should be that we don't have access to our reporting if we access it through the cloud. When I browse to "Myapps.Microsoft.Com", I see the list of "enterprise applications" that are made available to me, and CM Reporting is one of them. (A nice "touch" is that I get single-sign-on to this website because my device is Azure Ad joined)  

Upon selecting my CM Reporting app from the "MyApps" portal, I am notified that my device isn't able to access this resource.

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl19.PNG)

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl20.PNG)

As you can see from the screenshot above, I'm not allowed to access my "CM Reporting" app because I'm not compliant.  
**Note :** Clicking the "Device management portal" link actually opens up the ConfigMgr Software center on the Device compliance tab for more details.

In order to become compliant again, I have enabled bitlocker on my device. Once encryption is complete, we can validate from that same software center that we are now compliant !

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl21.PNG)

The same note from before applies. If you immediately check compliance after remediating, it could very well be that you are still being marked as non-compliant. Give it a bit more time to process all the changes.

And as my final result... I have now access to my Configmgr Reporting !!

![alt]({{ site.url }}{{ site.baseurl }}/images/CoMgmt/Compl22.PNG)

That's it folks ! I tried to cover an end-to-end scenario where co-management could be useful for you. I hope you enjoyed reading the series as I had writing them.  Let me know if you run into any issues or if you were able to replicate the setup in your own environment.

Take care !