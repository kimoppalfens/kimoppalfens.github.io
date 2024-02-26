---
title: "Unleashing Efficiency: Exploring the Benefits of Microsoft Endpoint Manager Co-Management and cloud attach"
header:
  overlay_image: tOSCCD32aR00aP02ZL-Logo Only.jpg
  teaser: OSCCD32aR00aP02ZL-Logo Only.jpg
author: kimoppalfens
date: 2024-02-05
categories:
  - SCCM
  - Configmgr
tags:
  - SCCM
  - ConfigMgr
---

Microsoft Endpoint Manager (MEM) Co-Management and ConfigMgr cloud attach are game-changers in the realm of mature IT management, offering a comprehensive suite of additional functionality to enhance device management capabilities of Microsoft Intune.

# Intro #
author: Kim Oppalfens

This innovative solution addresses the complex challenges faced by enterprises, bringing together Realtime script execution, application installation, CMPivot which offers realtime data collection, Autopilot integration with task sequences, custom inventory, flexible targeting of your deployments, Configuration Baselines, Software metering, customizable reporting, and the simple deployment of files without having to worry about securing the credentials needed for authentication. 

For managers that read this, I'll briefly add what not having these means throughout the rest of the post, but in short, it means your people will have to spend time coding workarounds and/or introduce cost to workaround these limitations. This in turn means knowledge transfer becomes harder as you can't just hire someone that knows the coded/introduced workarounds like you can with a popular off the shelf tool.

# Scope includes, but is not limited to **Azure AD Only** devices #

This post is heavily inspired on a breakout session Tom Degreef and I have been presenting on at several events.
One of the things heand I stress during this session is that this is geared towards both Azure AD (aka Entra ID) - Only devices as well as "Hybrid" joined devices.  A couple of strong misconceptions still survive in the Microsoft systems management world.

One of the strongest ones is the tie that people tend to make between Azure AD only devices and the available Microsoft Systems management solutions. It's true that Active Directory Group policies are not available to you when managing Azure AD only devices. Frequently this introduces the split up between "on-prem" tools and "cloud" tools. That split-up is confusing in this respect as it just defines where the service is most often offered from, it does NOT define what it can manage. So, repeat after me, 

> Microsoft Configuration Manager by itself and in a Co-Management setup are perfectly capable of managing Azure AD only devices.

The flow of an Azure AD only client authenticating against a Configuration Manager backend is documented clearly on [Micosoft's documentation page](https://learn.microsoft.com/en-us/mem/configmgr/core/clients/manage/azure-ccmsetup)

A second misconception that appears to be out there is that managing clients with Configuration Manager introduces the need for VPN style solutions to connect to the Configuration Manager backend. So, repeat after me once more,

> ConfigMgr can manage clients directly over the Internet, the method to do that, that is most future ready is, by introducing a Cloud Management gateway.

The same section of Microsoft's documentation introduces customers to the [Cloud Management gateway](https://learn.microsoft.com/en-us/mem/configmgr/core/clients/manage/cmg/overview)

The documentation for planning, setup and managing a CMG is quite extensive.

The Cloud Management Gateway does come at a cost, all be it a small one in most cases. There's a recurring monthly fee for the service and a cost for the consumed network traffic known as egress.


# Supercharge your Intune setup #
## The enhanced features offered by Co-Management and Cloud Attach for Windows Workstations ##

In this article, we delve into the ten key benefits of Microsoft Endpoint Manager Co-Management for workstations. All of these features complement what Intune can offer today, none of them prevent an organization from handling the needs that can be covered by Intune to be covered by Intune. Server workloads are not one of the features we list although they're equally a good reason to keep a ConfigMgr environment around.

For each feature we'll detail whether it needs Co-Management, ConfigMgr cloud attach and whether content is required. When content is required the solution massively benefits from yet another ConfigMgr cloud feature called the Cloud Management Gateway. The Cloud Management Gateway offers a cost-effective method for making ConfigMgr content available from anywhere. The need to be able to work from anywhere (Which most of the times means, work from home) is one of the things that has really taken of in the past several years and is considered by many to be a core requirement of modern work environments.

### Realtime Script Execution:
MEM Co-Management allows IT administrators to execute scripts on endpoints in real-time. This capability provides unparalleled flexibility in managing and troubleshooting devices. Whether it's resolving issues promptly or implementing critical updates swiftly, the real-time script execution feature empowers administrators to take immediate actions, reducing downtime and improving overall system health. This functionality can be run against an individual endpoint or against a collection of devices. When running against a single device it integrates with the Intune portal, running it against multiple devices aka bulk actions is a feature that, unfortunately requires us to move back to the ConfigMgr admin ui as Intune doesn't support that (yet).

- **NOTE**: This functionality is available with Co-Management and Cloud attach. 2 ways available to make ConfigMgr cloud capable.

### Realtime Application Installation:
The ability to install applications in real-time is crucial for maintaining a secure and up-to-date IT environment. With MEM Co-Management, administrators can initiate the installation of applications across devices seamlessly. This ensures that all endpoints are equipped with the latest software, enhancing security, and standardizing the user experience. Realtime in this respect means applications installs start within minutes. This unfortunately is only true for applications defined within Configuration Manager. (People might recognize this functionaly from an Ignite demo years ago.) These application do however appear in the Intune Company portal app alongside the Intune deployed applications.

- **NOTE**: This functionality requires the application sources to be downloaded, as a result this functionality relies on Co-Management being enabled and ideally having a Cloud Management Gateway configured.

### CMPivot:
CMPivot is a powerful feature that allows administrators to query and retrieve real-time information from devices. This functionality enables rapid assessment and response to security threats, performance issues, and compliance concerns. By leveraging CMPivot, organizations can proactively address potential problems, strengthening their overall security posture. This functionality allows Systems Management admins to provide their management with answers to information around the workstation population in a matter of minutes. Not having this option might introduce delays in decession making or force decission making with less information.

This functionality can be run against an individual endpoint or against a collection of devices. When running against a single device it integrates with the Intune portal, running it against multiple devices aka bulk actions is a feature that, unfortunately requires us to move back to the ConfigMgr admin ui as Intune doesn't support that (yet).

Microsoft has released functionality to do this in Intune only as an add-on on top of Intune or as part of Intune suite. The Intune only functionality is currently limited to targetting individual devices.

- **NOTE**: This functionality is available with Co-Management and Cloud attach. 2 ways available to make ConfigMgr cloud capable.

### Autopilot Integration with Task Sequences:
The integration of Autopilot with task sequences in MEM Co-Management streamlines the deployment and provisioning of devices. This ensures a consistent and efficient onboarding experience for end-users. The automated processes reduce manual intervention, minimizing errors and enhancing overall IT productivity. This offers a number of options not available with Intune only and Autopilot, like controlling the order of application installs and embed scripts directly in the process while making sure they finish by the time the Autopilot process has completed. Tasksequences equally don't have the Windows Autopilot limit that prevents organizations from mixing MSI and setup.exe based installations.

- **NOTE**: This functionality typically requires application sources to be downloaded, as a result this functionality relies on Co-Management being enabled and ideally having a Cloud Management Gateway configured.



### Custom Inventory:
MEM Co-Management offers the capability to gather custom inventory information from devices. This flexibility allows organizations to tailor data collection to meet specific business needs. Custom inventory enhances the depth of insights available to administrators, facilitating more informed decision-making and strategic planning. All of the Inventory can then be used in grouping devices in collections. ConfigMgr native inventory and custom inventory integrate into the Intune Portal and can be consumed from there.

- **NOTE**: This functionality is available with Co-Management and Cloud attach. 2 ways available to make ConfigMgr cloud capable.

### Flexible Targeting:
The flexibility in targeting devices based on various criteria is a standout feature of MEM Co-Management. Administrators can target specific groups of devices or users, enabling a granular approach to management. This ensures that policies, updates, and configurations are applied precisely where needed, avoiding unnecessary disruptions to unaffected devices. This flexible targeting is done using a rules based system that fully integrates with the custom inventoy feature described above. The ConfigMgr groups (aka collections) created this way sync up with Azure Ad groups. This makes sure that this flexibility is available to both ConfigMgr as Intune based deployments.

- **NOTE**: This functionality is available with Co-Management and Cloud attach. 2 ways available to make ConfigMgr cloud capable.

### Configuration Baselines:
Configuration Baselines provide a standardized method for evaluating and enforcing compliance across devices. MEM Co-Management allows administrators to define and implement configuration baselines, ensuring that devices adhere to organizational policies. This consistency enhances security and reduces the risk of non-compliance. Baselines have a number of "providers" of which setting registry keys is just one. This allows the setting and maintaing of specific values in custom registry keys without having to adhere to script authoring and/or figuring out a way to re-validate the value hasn't changed.

- **NOTE**: This functionality is available with Co-Management and Cloud attach. 2 ways available to make ConfigMgr cloud capable.

### Software Metering:
Software metering capabilities in MEM Co-Management empower organizations to monitor and control the usage of applications. This feature helps in optimizing software licensing costs by identifying underutilized or unused applications. By gaining insights into software usage patterns, administrators can make informed decisions regarding licensing and resource allocation.

- **NOTE**: This functionality is available with Co-Management and Cloud attach. 2 ways available to make ConfigMgr cloud capable.

### Customizable Reporting:
MEM Co-Management offers robust reporting capabilities that can be customized to meet the specific needs of an organization. Administrators can generate detailed reports on device compliance, security vulnerabilities, and application usage. Customizable reporting enhances transparency and accountability, supporting evidence-based decision-making. ConfigMgr reports and custom built reports can be made available over an Azure AD application proxy so that Managers and Sysadmins can access them from anywhere. (Regular end users aren't the only folks working from home.) 

- **NOTE**: This functionality is available with Co-Management and Cloud attach. 2 ways available to make ConfigMgr cloud capable. All you need here is for devices to be in your ConfigMgr database.

### Deployment of Software Distribution Packages:
Efficient software distribution is a cornerstone of effective IT management. MEM Co-Management facilitates the deployment of software distribution packages across devices, ensuring timely updates and patches. This capability streamlines the maintenance of a secure and up-to-date IT infrastructure. ConfigMgr and Intune both have abilities to deploy actual applications, but neither of their mechanisms are very well suited to just distribute a number of files, like templates.

One could script the copying of these files or embed them into a "fake" application, but both generate overhead and potential headscratching around protecting the authentication flow involved in this.

- **NOTE**: This functionality typically requires application sources to be downloaded, as a result this functionality relies on Co-Management being enabled and ideally having a Cloud Management Gateway configured.


# Microsoft's messaging #

We've received the feedback a number of times that this doesn't appear to be in line with Microsoft's longer term goals. We can't comment on what Microsoft's long term goals are, other than that we expect it's to make a profit, that's their responsibility. 

Microsoft has stated recently that Co-Management isn't an end-goal for them. They've equally mentioned that they're aware there's gaps and have now started a renewed effort with Intune Suite to close some of those gaps. We currently interpret the " *This is not our end goal.* " similar to what we expect from Microsoft on the identity front. We don't expect Microsoft wants to run 2 Identity Platforms (Active Directory Domain Services (onprem) and Azure AD/Entra ID (Cloud)) forever, but we don't expect either one to disappear in the short run.

When all those gaps will be closed and at what cost remains to be seen. This article details the situation as it is at the start of 2024, close to 9 years after Windows 10 was released or close to 14 years after the Intune service was introduced to customers.

# Conclusion #

In conclusion, Microsoft Endpoint Manager Co-Management stands as a comprehensive solution, **included in your Intune core license grant**,  that empowers organizations to navigate the complexities of modern IT management for **both Azure AD Only and Hybrid devices**. As one attendee at a conference put it, 

>*"Configuration Manager should be the last server in your Active Directory environment you shut down! Just about 5 minutes before you decommission your last Domain Controller."* 

Tom and I consider it a very viable opion in managing Windows endpoints (Workstations and Servers alike). From real-time script execution through customizable reporting to custom Inventory each feature contributes to a more efficient, secure, and responsive device management environment. By embracing MEM Co-Management, organizations can unlock a new level of agility and effectiveness in meeting the evolving demands of the digital landscape.

Mature Systems Management doesn't escape the need for people to work from home, nor does it escape any of the needs that we've all known for years if not decades in this field. In today's risky onlne world doing less systems management just isn't a viable option, the needs for a properly controlled endpoint have heightened over the years not lowered. 

In a world where the mantra is (still) "*Do more with less*" replacing a bunch of functionality that is readilly available with custom developed solutions just doesn't feel like something a mature organization would do.



