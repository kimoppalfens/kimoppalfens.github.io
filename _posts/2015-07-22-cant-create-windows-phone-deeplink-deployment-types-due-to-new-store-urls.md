---
title: "Cant Create Windows Phone Deeplink Deployment Types Due To New Store Urls"
author: OSCC
date: 2015-07-22
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

## Problem Description:  
  
## Update3:
This update has been closed as fixed on the connect page. If you're still seeing this issue please install the latest CU updates or upgrade to Configuration Manager current branch.

## Update2:  

The powershell script below returns the old url when you input the new url in the variable. The old url returned lets you create the app from the admin ui. Again, don't have a windows phone device enrolled in this environment, so can't test an actual deployment.  

```posh
$newWindowsDeeplinkURL='https://www.microsoft.com/en-us/store/apps/onefootball/9wzdncrfj4kx'  
#This is the url you can browse to new in internet explorer  

$oldwindowshDeeplinkURL  
=  
'http://www.windowsphone.com/en-us/store/app/'  
# this is the starting part of the old url, it is followed by the app name a guid  

$Links  
=  
Invoke-WebRequest  
https://www.microsoft.com/en-us/store/apps/onefootball/9wzdncrfj4kx  
|  
select  
-ExpandProperty  
Links  
|  
select  
href  
#get all links found on the new windows phone url  

foreach  
($link  
in  
$links)  
#cycle through all links  

{  

  
if  
($link.href.ToString()  
-imatch  
'ms-windows-store:navigate?appid=b[A-F0-9]{8}(?:-[A-F0-9]{4}){3}-[A-F0-9]{12}b')  
# find a link that starts with ms-windows-store and contains a guid behind appid=  

  
{  

  
$appguidrough  
=  
$link.href.ToString()  
|  
Select-String  
-Pattern  
'ms-windows-store:navigate?appid=b[A-F0-9]{8}(?:-[A-F0-9]{4}){3}-[A-F0-9]{12}b'  
#get the url that is a match for parsing  

  
$appguid  
=  
($appguidrough.ToString().Split('='))[1].Substring(0,36)  
# get the section of the url behing the appid= and take 36 characters from that point, this gives us the guid  

  
$appguid  

  
$appname  
=  
$newWindowsDeeplinkURL.Split('/')[6]  
# take the 7th parth from the new url, this gives us the appname  

  
$appname  

  
$oldwindowshDeeplinkURL  
+  
$appname  
+  
'/'  
+  
$appguid  
#concatenate the old starting url + the appname + the guid  

 

 

  
}  

 

}  
```
 

 

## Update1:   

## Early word is that the below procedure does let you create the app just fine, however installing the app would still fail. If you've tested this and actually deployed the app successfully let me know.  

One for my fresh new ConfigMgr MVP colleagues, Peter van der woude signaled an issue with creating new Windows Phone deeplink deployment types.   

It was signaled to the product team on Connect as well:   

  
  

This is caused by the windows phone store URL using a brand new url structured like this:   

  
  

whereas the old url looked something like this:   

  
  

![][1]  

The problem with this is that adminui.appmanfoundation defines a regular expression that validates the url input.   

public static Regex Winphone8DeeplinkUrlPattern;   

The new url no longer satisfies the regular expression and as such the wizard doesn't let you save your dt as it inspect the content location textbox before continuing.   

## Resolution/workaround   

As this is a UI thing, as can be identified by the fact that the issue resides in adminUI.appmanfoundation, I quickly assumed that Powershell would "not suffer" from this issue.   

And lo and behold, the following powershell commands will create a windows phone dt happily and add it to a previously created app with the name pswpdeeplinktest.   

(You still won't be able to edit it in the UI afterwards as the validation will kick back in).   

![][2]  

```posh
PS C:> Add-CMDeploymentType -WinPhone8DeeplinkInstaller -InstallationFileLocation 'https://www.microsoft.com/en-US/store/apps/Adobe-Reader/9WZDNCRFJ2GH' -ApplicationName 'pswpdeeplinktest'   
```
     
 

Keep in mind that Powershell and/or WMI can often "workaround" adminui limitations.   

Be carefull though as circumventing these limitations can be both a blessing and a curse.   

     
 

Enjoy.  
"The M in WMI stands for Magic"  
""Everyone is an expert at something" Kim Oppalfens - ConfigMgr Expert for lack of any other expertise  
System Center Configuration Manager MVP  
http://www.scug.be/blogs/sccm/default.aspx

http://www.linkedin.com/in/kimoppalfens

http://twitter.com/thewmiguy

[1]: https://kimoppalfens.github.io/assets/072215_1402_CantcreateW1.png
[2]: https://kimoppalfens.github.io/assets/072215_1402_CantcreateW2.png
