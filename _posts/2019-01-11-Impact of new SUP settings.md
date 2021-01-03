While checking all the software update configuration options due to the issue I encountered with the other changes, I also noticed a new option being available in the Supersedence Rules section of the Software Update Point Properties

![alt]({{ site.url }}{{ site.baseurl }}/images/SUP1.jpg)

As you can see, we can now individually control supersedence behavior for feature and non-feature updates. I assume most of you still use task sequences for feature upgrades, so the impact of this change is currently still very limited I think.

# Possible impact of changes #

When we configure a software update process, we usually create different rings/waves. An example of such a process can be like this :

Collection Name | Software available time | Installation Deadline
---|---|---
Technical Pilot | ADR Run time (ADR) | ADR + 1 day
Business Pilot | ADR + 7 days | ADR + 10 days
Production Wave 1 | ADR + 14 days | ADR + 28 days
Production Wave 2 | ADR + 16 days | ADR + 30 days

Since I live in Europe, we use the "offset" feature to evaluate the ADR 1 day after patch tuesday (in the evening).  

The result here is that the people in Production wave 2 will have their software updates enforced 31 days after patch tuesday at 9PM,so basically the installation starts on the 32nd day after patch tuesday.  
At least in most environments I encounter, most end users will still wait for the deadline to have their software updates installed, although they had 2 weeks notice...
