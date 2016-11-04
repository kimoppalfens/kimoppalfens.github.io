---
title: "PowerShell Logging using Log4Net in CMtrace format"
author: Kim Oppalfens
date: 2016-11-04
categories:
  - Powershell
  - 
tags:
  - Logging
  - CmTrace
---

This post goes wider than regular ConfigMgr stuff and is about adding logging to all your scripts.

# Intro #

As a ConfigMgr admin I've always been blessed to be working with a system that has extensive logging capabilities. A lot of my knowledge and troubleshooting skills, as a result, come from reading logfiles.

A while back, after writing yet another script, that could use decent logging. I set forward to find an easy way to add logging from the start of writing a new script. After some investigation I found a very popular logging framework from the .Net community called Log4Net [https://logging.apache.org/log4net/index.html](https://logging.apache.org/log4net/index.html "Log4Net")

# Framework features #
- Output to multiple logging targets (Console, File, Eventlog, etc...)
- Configuration in XML file external to the scripts itself
- Specify different Logging levels (None, Info, Warning, Error, Debug)
- auto-set logfile name to name of the script executing (*)
- auto-set location to ConfigMgr admin ui log folder, or sccm client log folder (*)
- use environment variables in log location
- Log in CMTrace format (*)

# Using the framework #
For those of you that just want to dive in, below you'll find the simple steps. Following these steps should provide you with all the bits needed to log output to console and a logfile.

1. Download the logframework from the Technet Gallery [https://gallery.technet.microsoft.com/Logging-solution-for-all-6894a554](https://gallery.technet.microsoft.com/Logging-solution-for-all-6894a554)
2. Unzip the Initialize-Logging.zip file to a location you chose.
3. Create your script
4. Copy/paste the Initialize-Logging function into your script, and execute the fuction Initialize-Logging, like this:


```posh
Initialize-Logging
```

You're script should now look like this:
![alt]({{ site.url }}{{ site.baseurl }}/assets/images/PowerShell-Logging-using-Log4Net-in-CMtrace-format-01.PNG)
{% raw %}![alt]({{ site.url }}{{ site.baseurl }}/images/P){% endraw %}


