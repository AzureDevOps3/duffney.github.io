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

### Execute Local Functions Remotely

To demonstrate how to do this, I'll use a famous Hell-World example. I have written an extremely simple function that writes
"Hello, World!" to the console. It is called MyFunction and has no parameters. I've loaded to into my local PowerShell session
and to run it remotely I will use Invoke-Command. Here is the trick, place a $ symbol out side the curly braces of the scriptblock.
Then put Function: inside the scriptblock before the name of the function as seen below.

{% highlight PowerShell %}
function MyFunction ()
{
    Write-Host 'Hello, World!'
}

Invoke-Command -ComputerName DC1 `
-ScriptBlock ${Function:MyFunction}
{% end highlight %}


![MyFunction](/images/posts/2016-8-9/MyFunction.gif "MyFunction")


### Using Parameters