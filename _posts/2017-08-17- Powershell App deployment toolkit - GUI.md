---
title: "Powershell App Deployment Tookit - GUI"
header:
author: Tom Degreef
date: 2017-08-17
categories:
  - SCCM
  - Configmgr
  - Powershell
  - Posh

tags:
  - SCCM
  - Configmgr
  - Powershell
  - Posh
---

GUI for creating applications using the Powershell App Deployment Toolkit, including creating Software ID Tags for easier License Management.

# Intro #

If you are working with ConfigMgr, you will probably create Applications once in a while to deploy to end-users machines :)

Although the current Application model is great and allows for much greater flexibility than we had with Packages/Programs , there are still situations where the Application model isn't sufficient.

A lot of the things that are not possible with the App model, are probably fixed now by writing custom scripts.

A very good alternative is to start using the [Powershell App Deployment Toolkit](http://psappdeploytoolkit.com/).

It is basically a (free) solution that allows you to standardize the way you deploy applications (and packages if you want to). It offers great flexibility and takes away most of the gaps that are left in the app-model.

However, if you have never used it before, it could be a bit overwhelming at first since there are so many different options available.