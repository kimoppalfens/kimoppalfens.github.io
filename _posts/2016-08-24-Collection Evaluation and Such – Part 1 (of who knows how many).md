---
title: "Collection Evaluation and Such – Part 1 (of who knows how many)"
author: Kim Oppalfens
header:
  overlay_image: logging1280x960.jpg
  teaser: logging512x384.jpg
date: 2016-08-24
categories:
  - Powershell
  - SCCM
tags:
  - SCCM
---
Another post that is long overdue, based on a session Jason Sandys and I delivered at MMS 2015. In that session Jason and I covered the internals of collections, and collection evaluation, much like we did about policies in this years May MMS event. At the end of that sessions we kind off promised to split the session up into different blog topics, as MMS sessions aren't recorded.

With the 2016 MMS and such we did have good reasons not to find the time to get around to posting these. (Yes, reasons, I am calling them reasons, they are not excuses, and I am sticking to that). However, after the Policies A through Z session I delivered with Tom Degreef I had an attendee step up to me to remind me about that promise. (Mental note: That's a serious downside of not having a speaker room at MMS, as a speaker you can't hide, and people step up to you and confront you with promises you haven't delivered on. I either need to be more careful on what I promise, or find a good disguise. Somehow thinking of a pink bunny custome, right now)

Ok, snapping out of mental pictures and the introduction, Collections is the topic at hand. The session started off by explaining why collections are this important. I am not going to cover that in this blogpost as you should either already know that, or you can download the powerpoint slides from the link above, and should be able to make sense out of that part from the slides and notes, but in short, collections are everywhere in System Center Configuration Manager

Collections have different rule types:

* Direct Membership
* Query based
* Include collections
* Exclude collections
* Device category (New in ConfigMgr Current branch)

Again, I won't cover the basics around those, but they're important later in this blog series when we start talking about collection evaluation.

This part of the lingo is important, and most likely less known to the public. Depending how you look at it, there are 3 or 4 types of collection evaluations.

## AdminUI based definition

If you take the AdminUI-based definition, then there are 3 collection evaluation types:

Again, this definition most likely doesn't require much more explanation than the names already imply.

## Colleval.log based definition

This is a different matter, and the lingo you'll see back in the colleval.log file (The single most important logfile regarding learning about collections).

In that definition there are actually 4 evaluation types:

* Primary
* Express
* Single
* Auxiliary

### Primary evaluation

Primary evaluation is the colleval.log equivalent of a full collection update that runs on a schedule. Now you might wonder why, if this is the same as a full collection the logs don't refer to this as a full collection evaluation rather than a Primary evaluation. Well, there is a pretty sensible explanation for that.

### Express evaluation

Express is the incremental evaluation that runs by default every 5 minutes (Configurable in Site Components) for every collection that you have configured for incremental evaluation. More details on explaining this evaluation will follow in part 2 and subsequent blogs, as explaining incremental evaluation takes time.

### Single evaluation 

This is a manual collection update for a collection that has no dependencies. The concept of collection 'dependencies' will follow later on.

### Auxiliary evaluation 

This is a manual collection update, but as you might guess at this point, is for a collection that does have dependencies.

### The Graph

You'll see colleval log refer to the word graph regularly. The graph in and of itself is not a collection evaluation type, but I'll explain the lingo here nonetheless as it's largely related. As I expect most of the readers of this blog already know Configuration Manager 2012 and beyond introduced a new concept of mandatory collection limiting when building your collections. This in turn result in a collection chain if you wish. In a more technical description, a collection A that is limited to a collection X actually depends on this collection X from an SCCM perspective.

Equally interesting in this part of the blogpost is the relation of a collection with its Include and Exclude rules. These are equally translated into dependencies. Yet the dependency is, from what I gathered from talking to people in my day job, and at conferences, slightly different than what most expect. Collection A with an include rule for Collection B, and an exclude for Collection C has 2 dependency relations. Being, collection A depends on collection B, and also depends on collection C.

![][1]

 

If you're still awake, and following, the reason all of this is important is that since Configuration Manager 2012 R2, any collection evaluation calculates "the graph". This graph, in my comprehension is sort of a map of collections that will be triggered for evaluation based on the collection evaluation that has set things in motion. Again, you should be aware by now that triggering a collection update a certain level in your collection chain/hierarchy triggers a collection update on all collections limited to the initiating collection, and so on throughout the entire chain. That part is well-known and documented by now. From a technical point of view though that statement really means that the graph/map is build based upon the collection-dependencies in your environment. As you can see in the picture above, and gather from the explanation, include and exclude rules are equally part of that graph. Now, you can take my word from that, or go on an investigation challenge and look through the vCollectionDepencyChain view, to verify all of this.

This roughly brings me to slide 14 of this session, and I'll sign out for now. Expect part 2, in less time than you all had to wait for part 1.


[1]: http://oscc1-public.sharepoint.com/Lists/Photos/082416_1317_CollectionE1.png