---
layout: post
title:  "Using Line Breakpoints in VScode to Debug PowerShell"
date:   2018-01-06 09:02:00
comments: true
tags: [PowerShell, visualstudio, visualstudiocode, vscode, debug, debugging, breakpoints, conditional breakpoints, function breakpoints]
modified: 2018-01-06
---

Debugging doesn't have to be limited to write-host messages in your PowerShell code. In this blog post you'll learn how to use the debugger within [Visual Studio Code](https://code.visualstudio.com/) to set line breakpoints allowing you to pause the code during execution, which then give you the ability to peer inside the scripts logic making it much easier to debug. I will also demonstrate how to use the debugger to manage your breakpoints by enabling or disabling them without completely removing them.

For this blog post you'll need two things.

* [Visual Studio Code](https://code.visualstudio.com/)
* [PowerShell extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell)

* TOC
{:toc}

# Line Breakpoints


Line breakpoints are the simpliest and most common of the breakpoints. So common in fact that when most people say breakpoint they are referring to line breakpoints. A line breakpoint is as the name implies a line in the code where the code should break or pause so you can debug. You can set line break points a few ways in VScode; hover your mouse to the left of the line number in the editor and left click or select the line you want to break on in the editor and hit F9.


## Setting Line Breakpoints: Mouse


One way to create line breakpoints is to simply hover your mouse to the left of the line number you want to break on and left click. Before you click a dim red circle will appear, but after you click a vibrant red circle is put in place. Also notice that a new breakpoint is added to the debugger panel in VScode, this panel will keep track of all the breakpoints you've placed in your code. If you don't see this panel click on the debug icon on the far left of the editor. It is the bug within a crossed out circle.


![linebreakpoint-mouse](/images/posts/UsingLineBreakpointsVScodeDebugPowerShell/linebreakpoint-mouse.gif "linebreakpoint-mouse")


## Setting Line Breakpoints: Keyboard F9


Another way to set line breakpoints is to use the keyboard shortcut `F9`. Simply select the line you want the breakpoint to be on and hit the F9 key. You can remove the breakpoint as well by hitting F9 again on the same line or on other lines with breakpoints. Again hitting F9 creates a breakpoint in the breakpoint panel of the left. A keyboard shortcut to open that panel is `Ctrl+Shift+D`


![linebreakpoint-keyboard](/images/posts/UsingLineBreakpointsVScodeDebugPowerShell/linebreakpoint-keyboard.gif "linebreakpoint-keyboard")


# Using Line Breakpoints


Visual studio code has two ways to debug. You can use the single file debugger or you can use the workspace debugger. The single file debugger is the default when you haven't setup a workspace, but what is a workspace? A workspace is a set of files that customize your editor. These customizations can be theme colors, formatting rules, and of course debugging settings. You can learn more about workspaces [here](https://code.visualstudio.com/docs/getstarted/settings). To determine which of the two debugging option you're using by looking at the top of the debugger panel. If it says _No Configuration_ next to the green start button, you're using the single file debugger. If it says anything other than _No Configuration_, then you're using the workspace debugger. The workspace debugger gives you several options for debugging, these settings change the way the debugger interacts with your code. More information about workspace debugging can be found [here](https://blogs.technet.microsoft.com/heyscriptingguy/2017/02/13/debugging-powershell-script-in-visual-studio-code-part-2/).


_You Can NOT debug unsaved files_


### Single File Debugger


![singlefiledebugger](/images/posts/UsingLineBreakpointsVScodeDebugPowerShell/singlefiledebugger.png "singlefiledebugger")


### Workspace Debugger

![workspacedebugger](/images/posts/UsingLineBreakpointsVScodeDebugPowerShell/workspacedebugger.png "workspacedebugger")


## Starting the Debugger


To enter or start the debugger you either click the green start button at the top of the debugger panel or you can hit the `F5` keyboard shortcut. After the debugger is started you'll notice a few things. Your integrated terminal terminal prompt changes, it nows has `[DBG]` on each line indicating it's inside the debugger. You'll also notice a yellow line in your editor, this is the line the debugger is currently on. A third indication is the debugger Action Panel at the top. This is what you use to navigate the code while inside the debugger. The debugger program will stop everytime it hits your breakpoint while executing your code's logic. The benefit to this is you get to see all the variables inside the code. To do that expand or look at the variable section inside the debugger panel. It contains several different variable scopes to navigate, making it easier to find what you're looking for.


![startdebugger](/images/posts/UsingLineBreakpointsVScodeDebugPowerShell/startdebugger.png "startdebugger")


## Managing Breakpoints


Creating and removing a few breakpoints is managable by just browsing to it and click F9 or the mouse on it, but that doesn't scale when you have say 10 or more throughout a large code base. The breakpoint section in the debugger panel gives you a few more was to manage the breakpoints. You can toggle on and off breakpoints one by one, you can add new ones, you can deactive\disable all breakpoints at once, and you can also remove all breakpoints which a single click. To see this options you have to hover your mouse on the breakpoint bar in the debugger panel.

### Breakpoint Section Options


* Toggle breakpoints on\off
* Create new breakpoints
* Deactive all breakpoints
* Remove all breakpoints


![breakpointoptions](/images/posts/UsingLineBreakpointsVScodeDebugPowerShell/breakpointoptions.png "breakpointoptions")

# Summary

This blog post introduced you to the debugger in Visual Studio Code, by demonstrating how to create line breakpoints and how to start and navigate the debugger panel. You also learned how to manage multiple breakpoints with some of the options available in the breakpoints section within the debugger panel. I hope you found this useful and that you start using the debugger for your daily debugging. It will make you a much more efficent coder in the end. This is only the tip of the iceburg, future things to learn would be how to use conditional or function breakpoints, watches, the call stack or how debugging a script is different than a function. If you're interested in learning all of this and more, you're more than welcome to check out my latest Pluralsight course [Debugging PowerShell in VS Code](https://app.pluralsight.com/library/courses/debugging-powershell-vs-code). If you do watch it or have watched it I want to thank you in advance! Please leave comments, feedback or questions below.
