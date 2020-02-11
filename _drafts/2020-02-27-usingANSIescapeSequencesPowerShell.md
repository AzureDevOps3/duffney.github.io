---
layout: post
title:  "Using ANSI Escape Sequences in PowerShell"
date:   2020-02-27 13:37:00
comments: true
modified: 2020-02-27
---

* TOC
{:toc}

# Introduction

When using ANSI within PowerShell there are two main parts; the `escape character` and the `escape sequence`. The escape character is used to indicate the beginning of a sequence and changes the meaning of the characters that follow the escape character. Most commonly escape characters are used to specify a virtual terminal sequence (ANSI escape sequence) that modifies the terminal. Escape characters are a standard of in-band signaling that control the cursor location, color, font styling, and other options within terminals and terminal emulators. ANSI escape sequences are often used with modifying command line prompt displays! 
Windows PowerShell doesn't have a built-in escape special character. Because of that you'd have to use `"$([char]27)"` to output a ASCII character representing an escape character. However, PowerShell now includes a [special character for escape](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_special_characters?view=powershell-7#escape-e) `` `e`` . To use the escape character you start a string with the escape character `` `e`` followed by an opening square bracket `` `e[``. Inside the square bracket is where you place the escape sequence. That escape sequence will determine how the terminal interpret the characters and acts accordingly. 

The best way to understand ANSI escape sequences is to break it down into its different parts. Using some ASCII art as an example you can break down the sequence `` `"`e[5;36m$asciiArt`e[0m"``  into its different parts. The sequence starts with the control sequence introducer `` `e[``.  The `` `e`` is the escape character and `[` is the introducer. What follows is the sequence. Each of the numbers within this sequence represent an argument. The number `5` represents an argument that makes the text within the sequence blink. Each argument must be separated by a semi colon, which is why you see a semi colon between 5 and 36. Values 30-37 represent different foreground colors. In this example `36` represents a foreground color of cyan. 

Next in the sequence is the letter `m` which represents a function. The function is called SGR (“Select Graphics Rendition”) and accepts several arguments which were define earlier in the sequence. What follows after the function is the text that will be displayed. In this example that is a here-string stored in a the variable `$acsiiArt` that contains the ASCII art for #PS7Now. At the very end of the sequence `` `e[0m`` is calling the SGR function again, but this time it is using the argument `0` to reset and turn off all the attributes defined in the first sequence. Putting the sequence back together again and running it within a terminal will result in the ASCII art #PSNow being displayed with a cyan font and flashing text. Now that you have a good understanding of ANSI escape sequences, let's take a look at what else can be done with them and have some fun!

```powershell
$asciiArt = @"
     __ __  ____ ___________   __             
  __/ // /_/ __ \ ___/__  / | / /___ _      __
 /_  _  __/ /_/ \__ \  / /  |/ / __ \ | /| / /
/_  _  __/ ____/__/ / / / /|  / /_/ / |/ |/ / 
 /_//_/ /_/   /____/ /_/_/ |_/\____/|__/|__/  
                                              
"@

Write-Output "`e[5;36m$asciiArt`e[0m";
#Write-Host "`e[5;38;5;40m$n `e[0m"
```
https://notes.burke.libbey.me/ansi-escape-codes/

# _insert gif here of blinking #ps7now_

_Blinking is determined by your terminal, it might not support it._

# Text Styling

ANSI escape sequences support a few different text styling options; bold, underline, and invert. Text styling with ANSI escape sequences are one of the more basic modifications you can make to the terminal's text. Each of the styles has a reset argument that will turn off the attribute it enabled. This is important to know, because if you do not reset the attribute all future inputs will have that style applied.

|Style|Sequence|Reset Sequence|
|---|---|---|
|Bold | \`e[1m | \`e[22m
|Underlined| \`e[4m | \`e[24m
|Inverted| \`e[7m | \`e[27m
|Reset all | | \`e[0m

## Bold

```powershell
$text = '#PS7Now'
Write-Output "`e[1m$text";
```

## Underline

```powershell
$text = '#PS7Now';
Write-Output "`e[4m$text";
```

## Invert

```powershell
$text = '#PS7Now';
Write-Output "`e[7m$text";
```

## Resets

All of the previous examples do not include a reset sequence. If you do not include a reset sequence the ANSI attributes you applied will continue to apply to all future text your code outputs to the screen. In PowerShell the command or script scope limit its impact, but it does effect the output of your code. Running the below code snippet, you'll notice that the text following #PS7Now is still bold and inverted. It will also change the first new line return of your terminal prompt.

```powershell
$text = '#PS7Now'
Write-Output "`e[1;7m$text, ANSI not reset, text is still bold and inverted"
```

When using ANSI reset sequences you have two options. You can either reset the attribute individually or reset all of them. Using the table above you know that the bold argument is `1` and the invert argument is `7`. Combining them you can create the escape sequence `` `e[1;7m``. This sequence will style the text bold and invert the foreground and background colors. To reset just the invert you would use the sequence `` `e[27m``. The text after that sequence would remain bold, but not inverted. To reset just the bold attribute you would use another sequence `` `e[22m``. That would remove all the attributes applied by the first sequence and the text would no longer be inverted or bold. Another option available if you do not need to keep any of the attributes enable is to reset all of them at once. The sequence to reset all attributes is `` `e[0m``.

```powershell
$text = '#PS7Now'
#reset styles individually
Write-Output "`e[1;7m$text `e[27m Not Inverted `e[22mNot Bold or Inverted";
#reset all
Write-Output "`e[1;7m$text`e[0m Not Bold or Inverted";
```

# Moving the Cursor

ANSI sequences can do more than just modify the style of text. ANSI also supports cursor movement. While I'm not sure of the practical usage of this in a PowerShell script, it is fun to play around with. Using `` `e[1m$pwd`e[2A;sleep 5`` as an example sequence you can see the curious position change. As you learned perviously the first sequence `` `e[1m$pwd`` is bolding the font and then outputting the variable `$pwd`. The second sequence `` `e[2A`` is what is moving the cursor position. The number `2` is defining how many positions to move the cursor and `A` is an ANSI function for cursor up. Normally this would happen so fast you wouldn't be able to see it. To take care of that the `sleep 5` is pausing the output for 5 seconds so you can see the cursor move.

_Learn more about ANSI cursor positioning [here](http://ascii-table.com/ansi-escape-sequences.php)._

```powershell
"`e[1m$pwd`e[2A";sleep 5
```

# Viewport Positioning "Scrolling"

Using the `S` ANSI sequence you can also use scrolling to add padding to the text that is output. The sequence `` `e[3S$text`e[S3`` will fill new lines in from the bottom of the screen. The value `3` is the number of lines that will be filled in. There is a scroll down sequence as well. You can read more about viewport positioning [here](https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#viewport-positioning).

```powershell
#Scroll Up (add 3 lines of space top & bottom)
$text = '#PS7Now'
Write-Output "`e[3S$text`e[3S"
```

# Cursor Positioning & Text Modification

ANSI escape sequences can also modify the cursor position. While the possibilities are endless, one way it can be used is to save and restore cursor location. To save the current cursor position you'll use the sequence `s` after the control sequence introducer. This will not modify the output of the string in anyway, it simply saves the position for restoring later. After the variable `$text` outputs #PS7Now you'll use the restore sequence which is `u`. This will place the cursor at the `#` character. With the cursor at that location you can use a text modification to delete the # at the beginning. The ANSI sequence for deleting a character is `P`. The `1` before the P is the number of characters that will be deleted. Running the below line of code will result in the `#` character being deleted. Simple, silly, but yet fun!

```powershell
$text = '#PS7Now'
Write-Output "`e[s$text`e[u`e[1P"
```

_Read more about [Text Modification](https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#text-modification) and [Cursor Positioning](https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#cursor-positioning)._

# 	Erase Display:
"`e[2J`e[1;38;5;93m$pwd.path `e[0m normal test"

# delete what you did
"`e[s`e[1;38;5;93m$pwd.path `e[0m normal test`e[u`e[K"

* erase line
* erase display

_april fools place erase display in a co-workers profile_

_read more [ANSI Escape sequence](http://ascii-table.com/ansi-escape-sequences.php)._

# Colors

## Basic 16-Color Foreground & Background

## 256-Color Foreground & Background

"`e[1;38;5;93m$pwd.path `e[0m normal test"

### How all 256 colors in the terminal 

```powershell
# 256-Color Foreground & Background Charts
$esc=$([char]27)
echo "`n$esc[1;4m256-Color Foreground & Background Charts$esc[0m"
foreach ($fgbg in 38,48) {  # foreground/background switch
  foreach ($color in 0..255) {  # color range
    #Display the colors
    $field = "$color".PadLeft(4)  # pad the chart boxes with spaces
    Write-Host -NoNewLine "$esc[$fgbg;5;${color}m$field $esc[0m"
    #Display 6 colors per line
    if ( (($color+1)%6) -eq 4 ) { echo "`r" }
  }
  echo `n
}
```

# Basic Prompt Modification with ANSI


_advanced prompt resources below_