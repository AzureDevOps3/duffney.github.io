---
layout: post
title:  "Writing Markdown from the Terminal with PowerShell"
date:   2020-02-26 13:37:00
comments: true
modified: 2020-02-26
---

* TOC
{:toc}

# Introduction

# Sources

[PowerShell and Markdown](https://ephos.github.io/posts/2018-8-1-PowerShell-Markdown#in-the-console)

[tip_colors_and_formatting](https://misc.flogisoft.com/bash/tip_colors_and_formatting)

[Console Virtual Terminal Sequences](https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#screen-colors)

[How To Use ANSI/VT100 Formatting in PowerShell](https://powershell.org/forums/topic/how-to-use-ansi-vt100-formatting-in-powershell-ooh-pretty-colors/)

[About Special Characters](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_special_characters?view=powershell-7#escape-e)

[The (Mostly) Dependency Free PowerShell Prompt - Part 1 ](https://ephos.github.io/posts/2019-6-24-PowerShell-Prompt-1)

[ANSI escape code](https://en.wikipedia.org/wiki/ANSI_escape_code)

[Bash tips: Colors and formatting (ANSI/VT100 Control sequences)](https://misc.flogisoft.com/bash/tip_colors_and_formatting#terminals_compatibility)


http://ascii-table.com/ansi-escape-sequences.php

https://wiki.archlinux.org/index.php/Bash/Prompt_customization

_output Shapecatcher_

https://unicode-table.com/en/#0025

### messing around

```powershell
# 	Erase Display:
"`e[2J`e[1;38;5;93m$pwd.path `e[0m normal test"

# delete what you did
"`e[s`e[1;38;5;93m$pwd.path `e[0m normal test`e[u`e[K"

# move up cursor
"`e[1;38;5;93m$pwd `e[0m normal test `e[2A";sleep 5

# add date
"`e[1;38;5;93m$pwd "+(get-date -f d)+"`e[0m normal test"

# blink
"`e[1;38;5;93m$pwd `e[0m `e[5mnormal test"

```

```powershell
"`e[1;38;5;93m$pwd.path `e[0m normal test"
```

```powershell
Set-MarkdownOption -Code '[38;5;220m';Show-Markdown ./2020-02-26-writingMarkdownfromtheTerminalwithPowerShell.md
```