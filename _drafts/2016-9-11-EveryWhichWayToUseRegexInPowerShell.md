---
layout: post
title:  "A Practical Guide for Using Regex in PowerShell"
date:   2016-9-11 09:02:00
comments: true
tags: [PowerShell, Regex, RegularExpression]
modified: 2016-9-11
---

### Introduction

In this blog post you'll learn severals ways to use regular expression from within PowerShell. You've most likely used some of these
techniques before. Such as the *-match* operator or the *select-string* cmdlet, but probably weren't aware you were using regular epxression.
This post will not teach you how to craft complex regular epxressions. Instead it focuses on how to use them in PowerShell to find matches, replace
text, and to split on matches.

### -match Operator

The -match operator matches a string with regular expression. It returns a true or false statement indicating whether or not a match was found. Then it
stores all the matches found in a variable called $matches. A simple example of this is below, I'm matching the word Administrator from a distinguished name
from Active Directory. This example might not seem like it's using regular expression because it's using only literal characters *Administrator*, but it is.

{% highlight PowerShell %}
'CN=Administrator,CN=Users,DC=wef,DC=com' -match 'Administrator'
{% endhighlight %}

![matches](/images/posts/2016-9-11/matches.gif "matches")

Let's make it a little more regex looking by replacing the literal characters *Administrator* with some regex metacharacters and a subexpression. I'll use the regex
expression *CN=(\w+)*. The C and N are still literal characters matching a captial C and a captial N, but \w is a metacharacters that matches any word character. The + sign is
another metacharacters that means to match one or more times. The parenthesis are used to capture the match found by \w+, in this example Administrator. The benefit
of using metacharacters is it makes the expression more dynamic just like paramters do for functions.

{% highlight PowerShell %}
'CN=Administrator,CN=Users,DC=wef,DC=com' -match 'CN=(\w+)'
{% endhighlight %}

![matchesmetachars](/images/posts/2016-9-11/matchesmetachars.gif "matchesmetachars")

The -match operator has a few different versions you should be aware of. By default PowerShell is case-insensitive so there is a case-sensitive version of the -match
operator -cmatch. There are opposites of both of these operators -notmatch and -cnotmatch. I won't cover all of these variants, but it's worth taking a look at the
-notmatch operator. Sometimes it's easier to say what you do not want than what you do want. The below example demonstrates this by getting a list of services where the name 
of the service does not match a digit. 

{% highlight PowerShell %}
Get-Service | where Name -NotMatch '\d'
{% endhighlight

![notmatch](/images/posts/2016-9-11/notmatch.png "notmatch")

### -replace Operator

Matching text is always the first part of using regular expression and it's useful, but most of the time you want to do something with that match. Using the same example
as above I can use the -replace operator instead of -match to replace the entire string with what I wanted to match. The expression looks like this *'CN=(\w+).\*','$1'*.
I'm using a subexpression () to capture the match 'Administrator'. Matches found within subexpression are stored in variables. In PowerShell they start at the number 0 and increment
by one for each subexpression used. That is why you see the *'$1'* in the expression. If you're not familair with -replace the synatx is *'this' -replace 'this','that'* which replace
the word this with that. Relating that to our expression *'CN=(\w+).\** matches the entire distinguished name and *'$1'* replaces it with the match found in the first subexpression. In
this case 'Administrator'.

{% highlight PowerShell %}
Get-Service | where Name -NotMatch '\d'
{% endhighlight

![replace](/images/posts/2016-9-11/replace.png "replace")

### Select-String cmdlet

Select-String searches for text and text patterns in either a string or a set of files. I use it mostly for looking through scripts I've written to identify which scripts
contain keywords I'm looking for. For example I could use it to search a folder for all scripts that are using Active Directory cmdlets with the verbs Get and Set. To do that, I 
could craft a simple expression that looks for a few verbs Get and Set followed by -AD then a series of words. That expression might look something like this *[GS]et-AD\w+*.
*[GS]* is a character class, but simply put it means allow G or S as the first character.

{% highlight PowerShell %}
Select-String -Pattern '[GS]et-AD\w+' -Path *.ps1 -List
{% endhighlight

![selectstring](/images/posts/2016-9-11/selectstring.png "selectstring")

### Switch Statements

Another way to use regex in PowerShell is within a switch statement. In the below example I'm using regular expression to validate user names in Active Directory. There are
three statements one validates service account names, another validates admin accounts and the last one identifies invalid characters. You could perhaps take this logic and 
add it to some PowerShell automation that creates user accounts. Each of these switch statements could execute different processes for creating Active Directory accounts. 

{% highlight PowerShell %}
$UserNames = 'svc_jenkins','duffney.admin','<jduffney>'

foreach ($UserName in $UserNames) {
    switch -regex ($var) {
        'svc_[^"/\\\[\]:;|=,+\*\?<>*]+' {"service account [$UserName]"}
        '[^"/\\\[\]:;|=,+\*\?<>*]+\.admin' {"admin account [$UserName]"}
        '["/\\\[\]:;|=,+\*\?<>*]+' {"[$UserName] contains invalid characters"}
    }   
}
{% endhighlight

### Regex Object

The previous examples of using regular expressions in PowerShell have been integrated into cmdlets a