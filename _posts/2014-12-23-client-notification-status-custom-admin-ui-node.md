---
title: "Client Notification Status Custom Admin Ui Node"
author: Kim Oppalfens
date: 2014-12-23
categories:
  - SCCM
tags:
  - SCCM
  - SDK
  - AdminUI
---

This blog post is a follow-up to my tweet around figuring out how-to create custom nodes in the Configuration Manager admin UI.

### Summary

The sample at hand will be based on displaying Client Notification status for clients based on the new Client Notification functionality added in ConfigMgr SP1. I'll split this post up in 2 sections, one for those of you that are interested in adding the Client Notification Status Node, and one for those that are interested in building their own node(s).

### The client notification status node explained

The client notification status is based on the feature added to System Center Configuration Manager 2012 SP1. The feature is also known as fast channel internally at Microsoft or the Big Green Button. You can't talk about this feature without mentioning the thoroughly detailed blog on the topic by Randy Xu. You can read up on that here: 

In the Q&A one of the question's is: Can I see the online status of clients from the Configuration Manager console? The answer to that has been, Not currently.

So that's what I wanted to solve as I was looking for a sample node I could create. The client notification status returns results from the SMS_CN_Clientstatus as mentioned in the blog post. (Funny side note, if you ask me, is that I actually found the blog post based on searching for this class, as I had discovered the class while casually browsing WMI. And yes, I am aware I have a problem :-))

The node has 5 columns, **client computer name**, **Online Status**, **Channel Type**, **LastStatusTime**, **ServerID**

#### NetBIOS Name

The Client computer name is self-explanatory.

#### 

#### Online Status

Online status is the info I really cared about. As that tells you whether client is "Online" Now what does Online actually mean in the context of Client Notification Status? Does that mean the machine is turned on and on the network? Well, the answer to that is "It depends". When we thoroughly read the aforementioned blog post there's a couple of caveats.

1. There's a random sleep timer of 10 minutes when starting a machine before it contacts the Client Notification Server (So the machine could be online although it is reported as offline because it hasn't reported in just yet. 
2. Client Notification Server is expecting some form of communication every 20 minutes, which means that worst-case a client could be reported as Online, whereas it actually went offline 19 minutes ago. 

So, to summarize this information isn't really real-time, although it does give you a pretty good indication.

#### Channel Type

The Client Notification Status feature can work over TCP (Port 10123 by default) or HTTP. There's some interesting things to note here as well. Most importantly, the thing that sets Client Notification Status apart from different right-click actions, is the direction of the network traffic. The Fast Channel communication channel is opened from the client towards the "Notification Server or BGB Server". Now, the Notification Server isn't a separate role you get to install, every management point post SP1 automagically becomes a BGB server as well. The interesting bit here is the direction, from a firewall perspective opening up communications in one direction or another can be a big deal. A lot of security folks will prefer not opening up any ports towards your clients.

The other important note in the Q&A section is that fast channel communications happens with the MP in the assigned site. So, when you have clients in a secondary site, they'll contact the BGB Server in the primary. This might make client notification non-workable in these type of environments unless you are prepared to let that firewall traffic pass.

Finally, there's the potential question on, why would I enable TCP when this can pass over https just as well? A question I won't answer here, a) because it's a why question :-) and b) because it is thoroughly answered in the product team's blog.

#### Last Status Time

Not entirely sure on this one, but my guess is this is updated whenever a client contacts its BGB server, so that would be on establishing the fast channel. My preliminary tests show out this happens on Client restart, yet not on successfully executing a client notification action.

#### 

#### Server ID

The Server ID is the actual BGB Server you are having a fast channel with. I am not positive on how interesting this info is, and neither whether this can be different from the management point you are connected to. I'd like for this to display the server name, however, haven't found a way to get the ID through WMI. I can get it from SQL but that would require special permissions and I am not really in favor of adding ui elements that bypass WMI.

So this is what it all looks like in the end: (The NetBIOS Name column entries have been masked to hide my lock of creativity in choosing computer names for my lab)

![image][1]

 

### Installing the Client Notification Status Node

Installing the custom console node is easy enough, especially for those that have installed extensions before. In essence it means placing some executable/dll in the adminconsolebin folder, plus adding the right xml to the AdminConsoleXmlStorageExtensionsNodes folder. The XML itself needs to be placed in a subfolder of this nodes folder. The subfolder needs the format of a guid, and the location in the admin ui depends on the guid used.

A lot has been written in the past, on how to figure out what guid to use. For this particular task, I just relied on adminui.consolebuider.exe, one of those nifty hidden tools in the Configuration Manager installation directory.

#### Step by Step

1. Download the ClientNotificationStatus.zip from here: 
2. Extract the zip file to a location of your choice. 
3. Copy the the client notification status.dll to the adminconsolebin folder of your admin ui console installation 
4. Create the Nodes  subfolder in AdminConsoleXmlStorageExtensions if it doesn't exist already 
5. Copy the guid folder (ec1eb040-7957-45c3-aad0-a0ef9afba98a) to AdminConsoleXmlStorageExtensionsNodes 
6. Restart the System Center Configuration Manager Admin UI 
7. The client status node in the Monitoring workspace should now contain a new sub view 

 

### Known issues / potential future enhancements

##### These have been pointed to me by a grumpy old man already (Not naming names here)

1. The node isn't limited to only display 1.000 entries by default 
2. The node doesn't have a search box like most other nodes 

##### Other items

* Show the BGB server name you're connected to. 
* see whether I can choose which columns to show by default 
* Include some form of RBA? 
