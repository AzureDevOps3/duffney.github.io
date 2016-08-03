---
layout: post
title:  "Create Jenkins JNLP Scheduled Tasks with PowerShell and DSC"
date:   2016-07-29 09:02:00
comments: true
tags: [PowerShell, Jenkins, DSC]
modified: 2016-07-29
---
### Applies to: Windows PowerShell 5.0

![Scheduled-Tasks.png](/images/posts/2016-07-29/Scheduled-Tasks.png "Scheduled-Tasks")

In this post you will learn, how to use Desired State Configuration to create a Jenkins JNLP scheduled task. A JNLP scheduled task is one way you can connect a Jenkins slave to the master
Jenkins instance. From my experience the JNLP scheduled tasks are more reliable than the Windows service option. This post first goes into how to create the JNLP scheduled task with PowerShell
and then moves into how to use that code inside the xScript DSC resource. The xScript resource in part of the [xPSDesiredStateConfiguration](https://github.com/PowerShell/xPSDesiredStateConfiguration) module.
It allows you to run PowerShell code or scripts inside DSC configurations without having to create a DSC resource. 


### Creating Scheduled Tasks with PowerShell

Before I could use the xScript resource, I had to figure out how to create a scheduled task with PowerShell. It was a little more complicated than I expected, but there were plenty of resources and blog posts that
helped me figure it out. Below is the code for creating the JNLP scheduled task with PowerShell. It starts of by creating a hash table $ActionParams. This hash table defines the execute, argument, and workdirectory
parameters for the New-ScheduledTaskAction cmdlet. Execute is the path to the java.exe, argument is the java command to run that connects the slave to the master Jenkins server and working directory is the location of
the slave.jar file. To learn about launching java web start slave agents with Windows scheduled tasks check out this [wiki page](https://wiki.jenkins-ci.org/display/JENKINS/Launch+Java+Web+Start+slave+agent+via+Windows+Scheduler).

After the $ActionParams hash table, I'm populating a variable called $Trigger which will set the trigger for the scheduled task. I've set it to begin at startup with a random delay timespan of 5 minutes. I found that if you
do not delay at startup the slave doesn't connect to the master consistently. Below the trigger I'm defining all the settings for the scheduled task. The scheduled task will not stop on idle, its restart interval is set to 1
minute, the restart count is set to 10 and it will start when available. The next setting took me awhile to find, $Settings.ExecutionTimeLimit = "PT0S" is unchecking the box next to "Stop the task if it runs longer than:" 
This is very important because you never want this task to end. Lastly I'm scheduling, registering and starting the scheduled task. Because the scheduled task has to be started by the user account that is set to run it, 
I have to use invoke command and supply the svc_jenkins account credentials. While the below code works some of you might not be comfortable storing clear text passwords in scripts. I wasn't either, read 
[Create Scheduled Tasks with Secure Passwords](http://duffney.io/Create-ScheduledTasks-SecurePassword) to learn how to avoid these clear text passwords. 

{% highlight PowerShell %}
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

$Task | Register-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent' -User 'svc_jenkins' -Password 'P@ssw0rd'

Invoke-Command -ComputerName slave -ScriptBlock {Start-ScheduledTask 'Jenkins JNLP Slave Agent'}  -Credential winops\svc_jenkins
{% endhighlight %}


### Using the xScript DSC Resource

As I mentioned before xScript is a great catch all DSC resource that allows you to run PowerShell code and scripts as a DSC resource without writing an entire DSC resource module for a single script. Just like a normal
DSC resource xScript has Get, Set, and Test functions. They are called GetScript, SetScript and TestScript and the do obvious things. GetScript is used to retrieve data about the script, SetScript is the function doing the work 
and applying the settings and TestScript is used to return true or false determining if it is in the desired state. Below are two examples of how to use the xScript DSC resource. The first example is cheating a little bit, by
setting the TestScript to $false I am ensuring that it executes the SetScript every time. GetScript is set to return nothing, so don't expect much from Get-DscConfiguration. This method is also problematic because once the DSC
Configuration runs a second time and it can't overwrite the first pass it will error out. 

Moving on to the second example, this example takes advantage to some extent of the GetScript and TestScript. It adds some logic to the TestScript to determine if it's in the desired state or not. GetScript will also return the
contents of the file specified. GetScript in the current version 3.12.0.0 is pretty limited out of the box and only works with one return value. If you want to add more acceptable key values you'll have to modify the schema.mof file
for the xScript DSC resource. Because scheduled task do not overwrite one another by default, I'll be using something similar to the second example. For more examples of the Script and xScript DSC resource check out these two blog
posts; [DSC: Script Resource GetScript](http://www.ultimaforsan.com/logs/2015/7/22/dsc-script-resource-getscript) and [DSC Script Resource](https://msdn.microsoft.com/en-us/powershell/dsc/scriptresource).


{% highlight PowerShell %}
xScript ScriptExample
{
    SetScript = { 
        $sw = New-Object System.IO.StreamWriter("C:\TempFolder\TestFile.txt")
        $sw.WriteLine("Some sample string")
        $sw.Close()
    }
    TestScript = { $false }
    GetScript = { #Do Nothing }          
}

xScript ScriptExample
{
    SetScript = { 
        $sw = New-Object System.IO.StreamWriter("C:\TempFolder\TestFile.txt")
        $sw.WriteLine("Some sample string")
        $sw.Close()
    }
    TestScript = { Test-Path "C:\TempFolder\TestFile.txt" }
    GetScript = { @{ Result = (Get-Content C:\TempFolder\TestFile.txt) } }          
}
{% endhighlight %}


#### Using xScript to Create Jenkins JNLP Scheduled Task

Next, I need to convert the PowerShell code from above into a DSC resource using the xScript resource. It's pretty straight forward, take all the code from above and place it inside the SetScript block.
Then create some simple logic for the GetScript and TestScript functions. I decided to return the state of the scheduled task with GetScript and TestScript simply validates that the scheduled task
exists. You might of already noticed, but this TestScript is pretty short sided. If I were to update the scheduled task and it already exists the next time it runs it won't get updated because TestScript
would of passed its evaluation. The logic for GetScript and TestScript can be as simple or as complex as you want or need it to be. 


{% highlight PowerShell %}
xScript NewJNLPScheduledTask {
    
    GetScript = {
        
        return @{
            Result=(Get-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent').State
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

        $output = $Task | Register-ScheduledTask -TaskName 'Jenkins JNLP Slave Agent' -User $using:User  -Password $using:Password

        Write-Verbose -message $output
    }

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
} 
{% endhighlight %}


### Jenkins JNLP Scheduled Task DSC Configuration

Here is the complete DSC configurations, it will create a scheduled task on a remote slave to connect to the Jenkins master with the Java web start command. Make sure the remote slave has Java installed, the xPSDesiredStateConfiguration
module available and that the slave.jar exists in the working directory specified by the scheduled task. I've also changed how I'm passing the user name and password to Register-ScheduledTask, I'm no longer supplying a clear text
password. On lines 20 and 21, I'm extracting out the user name and password from the credential object passed to the configurations via the configdata then providing those parameters to the Register-ScheduledTask cmdlet. 

{% gist bf7400da4c8a61a162f087fb8cd503a1 %}