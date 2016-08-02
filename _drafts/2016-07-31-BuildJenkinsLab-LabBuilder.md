---
layout: post
title:  "Build a Jenkins Lab With LabBuilder"
date:   2016-07-31 09:02:00
comments: true
tags: [PowerShell, Jenkins, DSC, LabBuilder]
modified: 2016-07-31
---
### Applies to: Windows PowerShell 5.0

New-Lab -LabPath F:\Jenkins -ConfigPath F:\Jenkins\Jenkins.xml -Name JenkinsLab
modify the xml
[download 2016 TP5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview?i=1#evaluation_4210)
Install-Lab -ConfigPath F:\Jenkins\Jenkins.xml -Verbose

mount iso to master
change source path D:\sources\sxs
log into master
configure jnlp port fix 7777
get agent deails from slave node settings 