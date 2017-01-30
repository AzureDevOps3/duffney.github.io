---
layout: post
title:  "Adding Credentials to Advanced Functions"
date:   2017-01-29 09:02:00
comments: true
tags: [PowerShell, Credentials, PSCredentials]
modified: 2017-01-29
---

Often times, you'll need to use cmdlets or command line utilites that do not support credential objects. Meaning they require
you to pass in the password in plan text. If you're like me you don't like this for two reasons. One,
it's annoying to have to provide a Username param and a Password param. Second, it makes you a twitch because
you're putting your password plain text is scripts. In this blog post I'll show you one of serveral ways you can add a credential param
to and advanced function even when the cmdlets within the advanced function don't support credential objects. 

### Adding a Credential Parameter

Adding a credential parameter is easy enough. Instead of using a Username and Password param you just add a single parameter called Credential. The
name of the parameter can be anything you want, but since most PowerShell cmdlet that support credential objects use that name I tend to stick with it.
