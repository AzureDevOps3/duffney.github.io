---
layout: post
title:  "Create Jenkins JNLP Scheduled Tasks with DSC"
date:   2016-07-29 09:02:00
comments: true
tags: [PowerShell, Jenkins, DSC]
modified: 2016-07-29
---
### Applies to: Windows PowerShell 5.0


In this post you will learn, how to use Desired State Configuration to create a Jenkins JNLP scheduled task. A JNLP scheduled task is one way you can connect a Jenkins slave to the master
Jenkins instance. From my experience the JNLP scheduled task is more reliable than the Windows service option. This post first goes into how to create a scheduled task with PowerShell
and then moves into how to use that code inside the xScript DSC resoruce. The xScript resoruce in part of the [xPSDesiredStateConfiguration](https://github.com/PowerShell/xPSDesiredStateConfiguration) module.
It allows you to run PowerShell code or scripts inside DSC configurations without having to create a DSC resoruce.


### Creating Scheduled Tasks with PowerShell

Before I could use the xScript resource I had to figure out how to create a scheduled task with PowerShell. It was a little more complicated than I expected, but there were plenty of resources and blog posts that
helped me figure it out. Below is the code for creating a scheduled task with PowerShell. I've customized it a bit, so allow me to explain a few of these settings. 

$ActionParams is a hash table that I will splat to New-ScheduledTaskAction. It specifies the execute, argument, and working directory parameters. Execute is the location of the java.exe, argument is the java command to [connect to the Jenkins master](https://wiki.jenkins-ci.org/display/JENKINS/Launch+Java+Web+Start+slave+agent+via+Windows+Scheduler), and working directory is the working directory I've defined for all my Jenkins slaves. After $Action I'm defining the Trigger, It wil occur at startup 
but after 5 minutes. Next I'm configuring the settings for the scheduled task. The task will not stop when it becomes idle, it restarts after 1 minute if it fails, attempts to restart 10 times and starts when available. $Settings.ExecutionTimeLimit = "PT0S" is 
unchecking the box next to "Stop the task if it runs longer than:" This is very important because you never want this task to end. Lastly I create the task and then register it so it will show up in Task Scheduler. Notice a big no no? Yep, I have the
password in clear text! Check out [Create Scheduled Tasks with Secure Passwords](http://duffney.io/Create-ScheduledTasks-SecurePassword) to learn how to get around that. I'll also address it later in the DSC configuration. 

{% highlight PowerShell %}
$ActionParams = @{
    Execute = 'C:\Program Files\Java\jdk1.8.0_102\bin\java.exe'
    Argument = '-jar slave.jar -jnlpUrl http://master/computer/Slave/slave-agent.jnlp -secret 37e93023c74ae1529db72342d511e508c5ec929138484c7b358be29b1196c7ed'
    WorkingDirectory = 'C:\Checkouts'
}

$Action = New-ScheduledTaskAction @ActionParams
$Trigger = New-ScheduledTaskTrigger -RandomDelay (New-TimeSpan -Minutes 5) -AtStartup
$Settings = New-ScheduledTaskSettingsSet -DontStopOnIdleEnd -RestartInterval (New-TimeSpan -Minutes 1) -RestartCount 10 -StartWhenAvailable -IdleDuration (New-TimeSpan -Days 20) 
$Settings.ExecutionTimeLimit = "PT0S"
$Task = New-ScheduledTask -Action $Action -Trigger $Trigger -Settings $Settings

$Task | Register-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent' -User 'svc_jenkins' -Password 'P@ss0wrd'
#Add start ScheduledTask
{% endhighlight %}

### Using the xScript DSC Resource

As I mentioned before xScript is a great catch all DSC resource that allows you to run PowerShell code and scripts as a DSC resource without writting an entire DSC resource module for a single script. Just like a normal
DSC resource xScript has Get, Set, and Test functions. They are called GetScript, SetScript and TestScript and the do obvious things. GetScript is used to retrieve data about the script, SetScript is the function doing the work 
and applying the settings and TestScript is used to return true or false determining if it is in the desired state. Below is the Sample_xScript.ps1 found on the [xPSDesiredStateConfiguration](https://github.com/PowerShell/xPSDesiredStateConfiguration/blob/dev/Examples/Sample_xScript.ps1) Github page. This provides you with a very simple example of how to use the xScript resource.

{% highlight PowerShell %}
Configuration xScriptExample {
    Import-DscResource -ModuleName xPSDesiredStateConfiguration

    xScript ScriptExample
    {
        SetScript = {
            $sw = New-Object System.IO.StreamWriter("C:\TempFolder\TestFile.txt")
            $sw.WriteLine("Some sample string")
            $sw.Close()
        }

        TestScript = { Test-Path "C:\TempFolder\TestFile.txt" }

        GetScript = { <# This must return a hash table #>
            @{
                Path = "C:\TempFolder\TestFile.txt"
                LineToWrite = "Some sample string"
            }
        }
    }
}
{% endhighlight %}

### Using xScript to Create Scheduled Tasks: Method One

In my opinion there are really two ways to use the xScript DSC resoruce. Method one is to use it like you would a normal script. System administrators and 
engineers typically write script that do something. They normally don't have testing logic or any advanced functions to return the status of what the script 
just did. There is nothing wrong with this approach and it does indeed get the job done. Below is how you would convert the above powershell code to a DSC 
resource using xScript. By leaving the GetScript blank and by having TestScript return False everytime, this DSC resoruce is treated like a script because
it will always execute. Even if the scheduled task is there and working when it checks in it will run the SetScript again. Again nothing wrong with this and
it does work.


{% highlight PowerShell %}
xScript NewJNLPScheduledTask {
    GetScript = {
        #Return nothing
    }

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

        $output = $Task | Register-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent' -User 'svc_jenkins' -Password 'P@ssw0rda'

        Write-Verbose -message $output
    }

    TestScript = {
        $false
    }
}                
{% endhighlight %}


#### Using xScript to Create Scheduled Tasks: Method Two


Method two involves adding some logic to TestScript and GetScript. GetScript must return a hashtable otherwise DSC won't be able to output that information.
You can use whatever means necessary in PowerShell to populate that hashtable but the output must be a hashtable. TestScript has to return a true or a false statement.
This typically consists of severl if statements determining if the script did what it was supposed to do. This can be as simple or complex as you'd like. The example
I have below is rather simple. GetScript gathers two pieces of information, it gets the scheduled task name and status then output that information in the form of a hashtable.
TestScript validates that the scheduled task exists by looking it up by its name. If you didn't immediatly notice there are some limitations to this. The first being if I changd any 
of the scheduled task settings and it already exists, the DSC configuration would not apply those settings because TestScript would result in True and skip the SetScript. The other is I hard
coded the name, so if you were to change the name the TestScript would aways fail. I did this to prove that it can be as simple or as complex as you want it to be.



{% highlight PowerShell %}
xScript NewJNLPScheduledTask {
    GetScript = {
        @{
            TaskName = (Get-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent').TaskName
            State = (Get-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent').State
        }                
    }

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

        $output = $Task | Register-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent' -User 'svc_jenkins' -Password 'P@ssw0rd'

        Write-Verbose -message $output
    }

    TestScript = {
        $ScheduledTask = Get-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent' -ErrorAction SilentlyContinue
        
        If ($ScheduledTask){
            Write-Verbose -Message "ScheduledTask Jenkins [JNLP Slave Agent] exists"
            $true
        } else {
            Write-Verbose -Message "ScheduledTask Jenkins [JNLP Slave Agent] did not exists calling SetScript"
            $false
        }
    }
}
{% endhighlight %}

### Using Environment Variables to Store Credentials


There is still one small problem with how I'm using the xScript DSC resource. The Credentials are stored in the configuration in clear text. This is more of a flaw with Register-ScheduledTask than with DSC,
but it still didn't sit well with me so I found away around it. The best method I found to deal with this issue is to create environment variables within the DSC configuration and pass those to the Register-ScheduledTask
cmdlet within the SetScript block. Below is a snippet of the DSC configuration that creates two environment variables, one for the user name and another for the password. $Node.SvcCredential is created from the configdata 
I'm then extracting out the password and username from the credential object and storing them in environment variables.

{% highlight PowerShell %}
Environment Password {
    Ensure = 'Present'
    Name = 'Password'
    Value = ($Node.SvcCredential.GetNetworkCredential().Password)
}

Environment UserName {
    Ensure = 'Present'
    Name = 'UserName'
    Value = $Node.SvcCredential.GetNetworkCredential().UserName
}
{% endhighlight %}


### Jenkins JNLP Scheduled Task DSC Configuration

Now that I've explained most of the code, here is the entire DSC configuration. There are a few extra things in here I haven't explained yet, but if your somewhat familair with DSC you'll understand the 
configuration just fine. I've included the LocalConfigurationManager block to change some of the LCM settings. The most important thing here is that I'm specifying the certificate thumbprint that will be used to 
decrypt the credentails in the .mof document. I'm also using the configdata to provide the nodename, certificate file and the credentails for the svc_jenkins account that will be used for the scheduled task. 

{% gist bf7400da4c8a61a162f087fb8cd503a1 %}