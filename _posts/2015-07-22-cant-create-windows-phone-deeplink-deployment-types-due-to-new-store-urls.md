---
title: "Cant Create Windows Phone Deeplink Deployment Types Due To New Store Urls"
author: Kim Oppalfens
date: 2015-07-22
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

This issue has been closed as fixed on the connect page. If you're still seeing this issue please install the latest CU updates or upgrade to Configuration Manager current branch.


The powershell script below returns the old url when you input the new url in the variable. The old url returned lets you create the app from the admin ui. Again, don't have a windows phone device enrolled in this environment, so can't test an actual deployment.  

```posh
$newWindowsDeeplinkURL='https://www.microsoft.com/en-us/store/apps/onefootball/9wzdncrfj4kx'  
#This is the url you can browse to new in internet explorer  

$oldwindowshDeeplinkURL = 'http://www.windowsphone.com/en-us/store/app/'  
# this is the starting part of the old url, it is followed by the app name a guid  

$Links = Invoke-WebRequest https://www.microsoft.com/en-us/store/apps/onefootball/9wzdncrfj4kx  
|  select -ExpandProperty  
Links |select href  
#get all links found on the new windows phone url  

foreach ($link in $links)  
#cycle through all links  
{  
  if ($link.href.ToString() -imatch 'ms-windows-store:navigate?appid=b[A-F0-9]{8}(?:-[A-F0-9]{4}){3}-[A-F0-9]{12}b')  
# find a link that starts with ms-windows-store and contains a guid behind appid=  
{  
  $appguidrough = $link.href.ToString() | 
Select-String  
-Pattern  
'ms-windows-store:navigate?appid=b[A-F0-9]{8}(?:-[A-F0-9]{4}){3}-[A-F0-9]{12}b'  
#get the url that is a match for parsing  

$appguid =  ($appguidrough.ToString().Split('='))[1].Substring(0,36)  
# get the section of the url behing the appid= and take 36 characters from that point, this gives us the guid  

$appguid  
$appname = $newWindowsDeeplinkURL.Split('/')[6]  
# take the 7th parth from the new url, this gives us the appname  
  
$appname  
$oldwindowshDeeplinkURL + $appname + '/' + $appguid  
#concatenate the old starting url + the appname + the guid  
}  
}  
```

Thanks for reading,
Kim

[1]: https://kimoppalfens.github.io/assets/072215_1402_CantcreateW1.png
[2]: https://kimoppalfens.github.io/assets/072215_1402_CantcreateW2.png
