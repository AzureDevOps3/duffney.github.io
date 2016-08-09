---
layout: post
title: Run Local Functions Remotely in PowerShell
comments: true
tags: [PowerShell]
modified: 2016-8-9 8:00:00
date: 2016-8-9 8:00:00
---

Have you ever had functions loaded into your local PowerShell session and needed to run then on a remote system? The typical solution
to this problem is to copy the code to the remote system and then load the functions on the remote system to use them. What if I told you
it is possible to run functions you have stored on your local machine and execute them remotely with Invoke-Command? This post will teach
you how to do this.