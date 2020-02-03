---
layout: post
title:  "Writing Markdown from the Terminal with PowerShell"
date:   2020-02-26 13:37:00
comments: true
modified: 2020-02-26
---

Introduction paragraph. This is some more text to fill some space.

* TOC
{:toc}


I've been using Ansible to manage a Windows environment for a little over a year now and I wanted to share a method of using Ansible that's helped me adopt it. Coming from a Windows background, I obviously have a strong PowerShell background and to be honest I struggled and still struggle sometimes using Ansible. The reason being, my default behavior is to open up a PowerShell prompt and start hacking away. Skilling up with Ansible has been somewhat difficult because I heavily lean on PowerShell when I need to do something quickly. What I've discovered is that by not taking those as opportunities to learn more Ansible I was slowing down my acquisition of Ansible knowledge. After realizing that I put some thought into how I could use Ansible more like I use PowerShell. Which made it easier for me to gravitate toward writing a playbook instead of writing a script. Using the method below I've started to shift my default behavior and have begun to enjoy learning Ansible a lot more.

* TOC
{:toc}

### Target All, All the Time

Ansible has the concept of groups. Those groups are defined in a host or inventory file. As the name implies groups are used to... well group machines together. Typically by type or function. For example, webserver or database. Being a PowerShell user, I've never really had to worry about an inventory per say or groups. I either knew the host names or the host name patterns and specified them or converted them to an array to be used with `Invoke-Command` or other cmdlets. Having to maintain an inventory for my purposes here was cumbersome and for that reason I default to using the `all` group in my playbooks. Unless I know the specific host names for the playbook I just leave the hosts section of the playbook set to `all` as shown below.

```
---
- hosts: all
```


### Define Connection Variables

 Because Ansible was developed to manage Linux systems first it's no surprise that it's default is to connect with ssh on port 22. That's a problem for those of us who manage Windows systems. Luckily it's a problem easily solved by defining some Ansible variables. There are A LOT of places you can define Ansible variables and this is one thing that tripped me up for awhile. Having these variables defined in a hostfile or group_vars location made Ansible seem to heavy. I wanted something light weight where I could write out a few tasks and target the systems I needed to without having to parse a hostfile or figure out which group had the correct variables. The solution to this problem is pretty simple. Just define the variables at the top of the playbook. Ansible offers several authentication options, which you can read more about on the [Windows Remote Management Authentication Options](https://docs.ansible.com/ansible/latest/user_guide/windows_winrm.html#authentication-options) wiki page. In this example, I'm using ntlm over WinRM with http. Previous to the variable `ansible_winrm_message_encryption`, you'd have to generate a self-signed certificate in order to setup a Windows host. Now, you don't have to worry about that. By specifiying `ansible_winrm_message_encryption: always` Ansible will enable message encryption and WinRM will be happy. You have a few different options for available and you can learn about about in the [Setting Up a Windows Host](https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html#setting-up-a-windows-host) wiki page. Special thanks to [Jeremy Murrah](https://twitter.com/JeremyMurrah) for pointing out the ansible_winrm_message_encryption [option to me](https://twitter.com/JeremyMurrah/status/1166783597065506821?s=20)!

```
- hosts: all
  vars:
    ansible_user: Administrator
    ansible_password: P@ssw0rd
    ansible_port: 5985
    ansible_connection: winrm
    ansible_winrm_transport: ntlm
    ansible_winrm_server_cert_validation: ignore
    ansible_winrm_message_encryption: always
```


### Sources 

[PowerShell and Markdown](https://ephos.github.io/posts/2018-8-1-PowerShell-Markdown#in-the-console)

[tip_colors_and_formatting](https://misc.flogisoft.com/bash/tip_colors_and_formatting)

[Console Virtual Terminal Sequences](https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences#screen-colors)

[How To Use ANSI/VT100 Formatting in PowerShell](https://powershell.org/forums/topic/how-to-use-ansi-vt100-formatting-in-powershell-ooh-pretty-colors/)

[About Special Characters](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_special_characters?view=powershell-7#escape-e)

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