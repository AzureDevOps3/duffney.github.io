---
layout: post
title:  "Doubling My Salary a PowerShell Story"
date:   2016-9-28 09:02:00
comments: true
tags: [PowerShell, Career, Story]
modified: 2016-9-28
---

[![powershelltriplesalary](/images/posts/2016-9-28/powershelltriplesalary.png "powershelltriplesalary")](https://twitter.com/CianAllner/status/780390924635500544)

I've seen tweets like this before and have even heard compelling stories about how a sys admin learned PowerShell and then got a 10% raise. Hearing stuff like this 
is always motivating to me, but it's not only motivating it's now a reality for me. The same day this tweets went out, I received an offer that has doubled my salary! Now,
I'll admit the offer didn't double my current salary, but it has doubled since I started learning and using PowerShell (3 years ago). The rest of this blog post is my PowerShell story.
I'm not writing this to toot my own horn, but to show you that investing in PowerShell is both rewarding and worth your time and effort. I'd also like to use this as an opporutnity
to regonize all of those who have helped me along the way. 

## The Beginning

Before I begin, let me just say I started in IT with the mindset of I won't want to code or program. I told myself I didn't want to stare at a screen of text all day and hack away. Well,
turns out that what I LOVE doing! Wish I had gone to school for computer science... anyway on with the story. My PowerShell story begins with my manager at the time requesting that I log into
a few machines each day to check their diskspace. I was to do this every morning first thing when I got in to the office. This wasn't bad for the first day or so, but after that it got really
really annoying. I knew there had to be a better way! Luckily, I had heard of PowerShell before and even used it a few times to create some mailboxes. So, at this point I knew the problem and I knew
I could probably accomplish it with PowerShell, but how? 

Like any sys admin since the invention of Google that is where I looked first. I found a lot of related stuff, but it just left me with more questions. What's the $var mean? How does the pipeline work?
what type of loop should I use? Those types of questions are harder to answer with Google, so I turned to the [Spiceworks Community](https://community.spiceworks.com). At the time I was a very active member in
the spiceworks community, but had stayed away from the PowerShell category. It was time to change that and I posted some of my very first PowerShell related questions. Within a few hours I had a functioning script called
[Simple Disk Space Report](https://community.spiceworks.com/scripts/show/2106-simple-disk-space-report) that would get the diskspace and even email it! How AWESOME is that! No more logging into servers! Needless to say 
that was the beginning of my addition of PowerShell and automation.

![simplediskspacereport](/images/posts/2016-9-28/simplediskspacereport.png "simplediskspacereport")

## Becoming a Scripter

It's worth mentioning that at this point in the story, I wasn't quite sold on the whole sys admin scripter thing. At the time I dreamed of becoming a CCIE and was studying for my CCNA. Man, was that pain in the A$$.
I started becoming a scripter when I worked on the service desk for a large enterprise. I was hired on as tier 2 support for network, Windows server and Citrix. I got a wide varity of requests, which I loved. It exposed me
to a lot of different technoliges and at scale. The tickets I didn't enjoy at first were the "Hey, we need these 300 computer objects disabled" or "please reclaim ownership of 6000 Citrix profiles". Almost everyone dreaded these tickets
because they were so time consuming and mundane. Long story short, another guy on my team and I started using PowerShell to complete these tickets. At first everyone thought we were God's of efficiency and then we became known as the
scripters.

I was only at the service desk for about 6 months before I accepted a position within the company to become their System Center Configuration Manager admin \ engineer. I didn't know it at the time, but this was a turning point in my
career. I spent the next year building up their application store within SCCM and revamping their operating system deployment. Since my manager and I were the only two SCCM admins automating wasn't an option it was a necessity.
I used PowerShell for everything I could to automate SCCM. Driver imports, application creation, application deployment, maintenance, etc.. I relied heavily on the PowerShell communites and a few engineers within the company
to improve my skills. It was at this point that I started reading everything I could on PowerShell. I started attending the local PowerShell user group every single month. I didn't understand half the presentations, but it was
motivating to see the potential of PowerShell. I also got to network with some great people, more on that later.

![sccmprojects](/images/posts/2016-9-28/sccmprojects.png "sccmprojects")

## Advocating PowerShell

After honing my PowerShell skills I wanted to give back. I began advocating and teaching PowerShell to practially anyone who would listen to me. I started setting up meetings with the service desk and desktop support teams to help
them learn PowerShell. I also started presenting at the local user group I was attending. Like most, I had a fear of public speaking, but I didn't let that stop me. _“Everything you want is on the other side of fear.” --Jack Canfield_.
It was at this point that I realized this is what I want to do with my career not networking, even though I had just passed my CCNA 6 months earlier... 

There's a saying _"it's not what you know but who you know"_. Well, there is a bit of truth to that statement. Presenting and networking at the local user group got me my next gig. I was happy at my current job and enjoyed alot
of the benefits of working at a large company so I asked for a lot of money. I interviewed for the position and it went well, but it was the person I had met at the user group who got me the job. He was advocating for me within the company
he was a PowerShell scripter himself and was a very valued person at the company. Luckily, for me his opinion carried enough weight to get me the job. This was truely life changing for me. My wife and I were expecting our first child
and we weren't sure how we could afford day care or if we should have her quit her job. This job allowed me to make up half of my wife's income. She was able to become a stay a home mom and I didn't have to worry about paying for 
the mortgage. Words can't express how grateful I am for that and it's because I started to learn PowerShell. That was the skill that I acuqired that made all of that line up perfectly.

![presenting](/images/posts/2016-9-28/presenting.png "presenting")

## Embracing DevOps

Just before I started this new job, I had just finished a book called [The Phoenix Project](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262592). This book was the start of a 2 year journy of understanding and
embracing DevOps. Once you've been scripting or automating for awhile you start to wonder; how do I better manage this code? is there a way to automatically execute code? how can I collaborate with others who write code? I found out
I could answer all of these questions by looking at what developers did. 