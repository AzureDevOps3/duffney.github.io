---
layout: post
title:  "Create Jenkins JNLP Scheduled Tasks with DSC"
date:   2016-08-07 09:02:00
comments: true
tags: [PowerShell, Jenkins, DSC]
modified: 2016-08-07
---
### Applies to: Windows PowerShell 5.0


In this post, you will learn how to create a Jenkins JNLP scheduled task on a Windows Server slave with Desired State Configuration.
In a pervious post I used PowerShell to create the scheduled task, I will now show you how to take that code and convert it to a DSC
configuraiton by using the xScript DSC resource. xScript is a DSC resource found in the [xPSDesiredStateConfiguration](https://github.com/PowerShell/xPSDesiredStateConfiguration) module.
It is a fantasic resource for running PowerShell code inside your DSC configurations

