---
layout: post
title:  "Create Jenkins JNLP Scheduled Tasks with DSC"
date:   2016-8-7 09:02:00
comments: true
tags: [PowerShell, Jenkins, DSC]
modified: 2016-8-7
---
### Applies to: Windows PowerShell 5.0


In this post, you will learn how to create a Jenkins JNLP scheduled task on a Windows Server slave with Desired State Configuration.
In a pervious post I used PowerShell to create the scheduled task, I will now show you how to take that code and convert it to a DSC
configuration by using the xScript DSC resource. xScript is a DSC resource found in the [xPSDesiredStateConfiguration](https://github.com/PowerShell/xPSDesiredStateConfiguration) module.
It is a fantastic resource for running PowerShell code inside your DSC configurations without having to write an entire resource.


### Download xPSDesiredStateConfiguration


Before you can use xScript, you first have to download the resource module. You can install the resource module with the following command.
-Force is being used incase you already have xPSDesiredStateConfiguration installed. It does not overwrite the existing module, simply adds 
the latest version. It's worth mentioning that the built-in PSDesiredStateConfiguration resource module has a similar DSC resource called 
Script that will work in most instances. However, xScript is the DSC resource that is and will continue to be updated by Microsoft and the
community. 

{% highlight PowerShell %}
Install-Module -Name xPSDesiredStateConfiguration -Force
{% end highlight %}


### GetScript


GetScript can be a bit tricky, because by default it must return a hashtable with keys that match the properties defined in the schema.
What does that mean?, you ask. Basically it has to look like the below code where it outputs a hashtable and the key is "return". Return is
the key that the xScript resource is looking for by default. You can modify the schema file and add more keys if you'd like. I chose to
return the state of the scheduled task when GetScript runs. 

{ % highlight PowerShell %} 
GetScript = {
    
    return @{
        Result=(Get-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent').State
    }
}
{% end highlight%}


### SetScript

Within the SetScript block is where I will place all of the code that is responsible for creating the scheduled task. It doesn't have to output
anything, but as a best practice I've placed a few Write-Verbose lines within the SetScript block. For more detail on the PowerShell code creating
the scheduled task reference this [post]().

{ % highlight PowerShell %}
SetScript = {     
    
    $ActionParams = @{
        Execute = 'C:\Program Files\Java\jdk1.8.0_102\bin\java.exe'
        Argument = '-jar slave.jar -jnlpUrl http://master/computer/Slave/slave-agent.jnlp -secret 37e93023c74ae1529db72342d511e508c5ec929138484c7b358be29b1196c7ed'
        WorkingDirectory = 'C:\Checkouts' 
    }

    $Action = New-ScheduledTaskAction @ActionParams
    $Trigger = New-ScheduledTaskTrigger -RandomDelay (New-TimeSpan -Minutes 5) -AtStartup
    $Settings = New-ScheduledTaskSettingsSet -DontStopOnIdleEnd -RestartInterval (New-TimeSpan -Minutes 1) -RestartCount 10 -StartWhenAvailable
    $Settings.ExecutionTimeLimit = "PT0S"
    $Task = New-ScheduledTask -Action $Action -Trigger $Trigger -Settings $Settings

    $output = $Task | Register-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent' -User $using:User  -Password $using:Password

    Write-Verbose -message $output
}
{% end highlight%}


### TestScript


TestScript is where you write the logic to determine if SetScript should run. I decided to keep it simple and just test whether or not the 
scheduled task existed. If it did it wouldn't execute SetScript, but if it didn't SetScript would execute. There is one obvious limitation to this
if for any reason the action, trigger or settings of the scheduled task were updated and it existed the changes wouldn't be applied because SetScript
would never run. You would have to remove the scheduled task first then rerun the DSC configuration.


{ % highlight PowerShell %}
TestScript = {
    $ScheduledTask = Get-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent' -ErrorAction SilentlyContinue
    
    If ($ScheduledTask){
        Write-Verbose -Message "ScheduledTask Jenkins [JNLP Slave Agent] exists"
        $true
    } else {
        Write-Verbose -Message "ScheduledTask Jenkins [JNLP Slave Agent] does not exists calling SetScript"
        $false
    }
}
{% end highlight%}


### The Complete DSC Configuration

Below, is the complete DSC configuration that will create a scheduled task that connects a Jenkins slave to the master server with JNLP. The last
thing I wanted to point out is how I'm passing the credentials to Register-ScheduledTask. I'm extracting out the credentials from the credentials
variable created within the configdata and storing them in the $Password and $User variables. I then have to use $Using to provide them to the -User
and -Password properties of Register-ScheduledTask. This way the password isn't stored in clear text within the DSC configuration.

{% gist bf7400da4c8a61a162f087fb8cd503a1 %}