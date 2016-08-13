---
layout: post
title: Splatting Parameters inside Advanced Functions
comments: true
tags: [PowerShell, ADCS, DSC, mof, Encrypting, Certificate, credentials, passwords, clear-text]
modified: 2015-11-7 8:00:00
date: 2015-11-7 8:00:00
---
#### Applies to: Windows PowerShell 3.0+

What is Splatting? In Windows PowerShell terms, splatting is a way of bundling parameters to send to a command.
[Don Jones 2011](https://technet.microsoft.com/en-us/magazine/gg675931.aspx) Splatting is most often used when 
providing parameters and their values to a cmdlet in the form of a hash table. A simple splatting example is provided below.

{% gist 5778099f75371cdbf707 %}

### Splatting Parameters Inside a Function

The main benefit to splatting the parameters inside the function is it provides another solution aside from using if and else if statements. The more parameters you provide with your function the more if or elseif statements you'd have to create. With splatting you simply add to your hashtable and then provide that as the parameters to your cmdlets inside the function. Below is a function that moves all the "members of" to the "members" section of an Active Directory group. I need to add two additional parameters to support running this against a different domain. The two parameters are $server and $pscredentials, I'll need both in order to run the script against a domain other than the one I'm executing the code in.

{% gist 650e6fd2f16710621cb0 %}

### Splatting Parameters Inside a Function in Action

Under this paragraph is the updated code that has multi domain support, it can be run against any domain with Active Directory Web Services running given you have the proper permissions in that domain. Two parameters were added $Server and $PScredentials, $Server is the Domain Controller you are targeting and $PSCredentails is used to store the credentials for that domain. Starting on line 37 the hashtables are built and line 49 and 50 add $PSCredential to the hashtables only if -PSCredential is used when the cmdlet is executed. Okay, we didn't fully get away from the if statement, but it still reduces the code quite a bit.

{% gist 804daede01d58cfa1118 %}

Shout out to the people at [DevOps Library](https://www.youtube.com/channel/UCOnioSzUZS-ZqsRnf38V2nA) for showing me this neat trick! 
Here is a link to an excellent video on [writing reusable PowerShell Code](https://www.youtube.com/watch?v=7jU15_7pPkY)!