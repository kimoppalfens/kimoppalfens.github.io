---
title: Products
author: OSCC
layout: single
excerpt: "Page not found. Your pixels are in another canvas."
sitemap: false
permalink: /tools/
author_profile: true
gallery3:
  - image_path: AdminPasswordRandomizer-CMTSStep.png
    alt: "placeholder image 2"
  - image_path: AdminPasswordRandomizer-CMTSStep2.png
    alt: "placeholder image 2"
---

## Admin Password randomizer SCCM Extension ##

Admin Password ramdomizer is a tool that randomizes the Local Administrator password and stores it in an encrypted manor in Active Directory and/or SCCM.

- Uses public / private keys to encrypt password
- Can update password in IBCM or new cloud management gateway environments
- Can update password directly in Active Directory on corpnet
- Allows protection of password using DACL on Active Directory property
- Use existing property or schema extension property as you see fit
- Use Active Directory auditing or SCCM auditing to monitor clear text password requests
- Control Character set including the ability to avoid ambiguous characters
- Integrates into SCCM, the number 1 systems management tool
	- Allows for easy scheduling
	- Monitoring the success of applying policies
	- Integration into your staging process


{% include gallery id="gallery3" caption="The Admin password randomizer extends SCCM with a tasksequence step, and an admin UI extension to request the passwords." %}


## Free Tools ##