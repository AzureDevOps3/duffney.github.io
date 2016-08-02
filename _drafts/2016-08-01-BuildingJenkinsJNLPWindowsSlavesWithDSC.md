---
layout: post
title:  "Building Jenkins JNLP Windows Slaves with DSC"
date:   2016-08-01 09:02:00
comments: true
tags: [PowerShell, Jenkins, DSC]
modified: 2016-08-01
---
### Applies to: Windows PowerShell 5.0


In this post you will learn, how to use Desired State Configuration to build a Jenkins Windows Slave that connects to the Jenkins Master with a JNLP scheduled task. 
I've foundthe JNLP scheduled task to be more reliable and stable than the Jenkins Windows service. I'm assuming you already have a Jenkins Master built and a 
Windows Server 2012R2 with PowerShell v5.0 installed or a Windows Server 2016 Technical Preview ready to be converted to a Jenkins slave. 

### Requirements

[Jenkins Master](https://dscottraynsford.wordpress.com/2016/04/18/install-jenkins-using-dsc-part-2/)
Windows Server 2012R2 PSv5 or Windows Server 2016 Technical Preview [Core or Full]


### Configure JNLP on the Master


