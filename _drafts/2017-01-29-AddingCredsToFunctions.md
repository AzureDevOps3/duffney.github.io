---
layout: post
title:  "Adding Credentials to PowerShell Functions"
date:   2017-01-29 09:02:00
comments: true
tags: [PowerShell, Credentials, PSCredentials]
modified: 2017-01-29
---

Often times, you'll need to use cmdlets or command line utilites that do not support credential objects. Meaning they require
you to pass in the password in plan text. If you're like me you don't like this for two reasons. First off,
it's annoying to have to provide a Username parameter and a Password parameter. Secondly, it makes you a twitch because
you're putting your password plain text is scripts. Another reason is you want the ability to run your function as a different user. Most likely one
with elevated access that you don't normally use. Oh, wait you're running your PowerShell session as domain admin right now? Please stop that...
In this blog post I'll show you serveral ways you can add a credential parameter to and function. Even when the cmdlets within the 
function doesn't support credential objects. 


### Adding a Credential Parameter


Adding a credential parameter is easy enough. Instead of using a Username and Password params you just add a single parameter called Credential. The
name of the parameter can be anything you want, but since most PowerShell cmdlet that support credential objects use that name I tend to stick with it.
Below, I'm using a function called Set-RemoteRegistryValue which is out of [The Pester Book](https://leanpub.com/the-pester-book). I've added the credential parameter
within the param block and also added the -Credential parameter to the `Invoke-Command` within the function. This will allow me to run this function with alternate
credentials. By using a default parameter value of `[System.Management.Automation.PSCredential]::Empty` I can make the credential param optional. Which means the
function will run when a credential is specified and when it is not. This is one of serveral ways to make it optional. Another would be to use splatting. If you're
interested in learning more about splatting inside functions take a look at this blog post [Splatting Parameters Inside Advanced Functions](http://duffney.io/Splatting-Parameters-Within-AdvancedFunctions)


_Tip_

"Caveat: Some cmdlets that accept a Credential parameter do not support/check for [System.Management.Automation.PSCredential]::Empty like they should. This should be treated as a bug by those cmdlet authors. There is a workaround though, and that is to use splatting. For example, if you want a solution that works for any cmdlet, you could do the following." Poshoholic [PowerShell.Org Form](https://powershell.org/forums/topic/using-credential-param-as-optional/)

{% highlight powershell %}
function Set-RemoteRegistryValue {
    param(
        $ComputerName,
        $Path,
        $Name,
        $Value,
        [ValidateNotNull()]
        [System.Management.Automation.PSCredential]
        [System.Management.Automation.Credential()]
        $Credential = [System.Management.Automation.PSCredential]::Empty        
    )
        $null = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            Set-ItemProperty -Path $using:Path -Name $using:Name -Value $using:Value
        } -Credential $Credential
}
{% endhighlight %}


### Adding a Credential Parameter with Splatting


As mentioned in the _Tip_ Poshoholic points out something very important, not all cmdlets support the use of `[System.Management.Automation.PSCredential]::Empty`. For that reason,
you might consider to play it safe and always add some splatting. I demonstrate what that looks like below, I've removed the `-ComputerName` and `-Credential` parameters from `Invoke-Command`
and instead pass them in by using `@splat` which is a hashtable with the key value pairs for the parameter and value of the parameter. The If statement in the code tests the $Credential
variable to see if $Credential is not equal to `[System.Management.Automation.PSCredential]::Empty` and if it is updates the hashtable to use `$Credential`.

{% highlight powershell %}
function Set-RemoteRegistryValue {
    param(
        $ComputerName,
        $Path,
        $Name,
        $Value,
        [ValidateNotNull()]
        [System.Management.Automation.PSCredential]
        [System.Management.Automation.Credential()]
        $Credential = [System.Management.Automation.PSCredential]::Empty        
    )
        
        $Splat = @{
            ComputerName = $ComputerName
        }

        if ($Credential -ne [System.Management.Automation.PSCredential]::Empty) {
            $Splat['Credential'] = $Credential
        }
        
        $null = Invoke-Command -ScriptBlock {
            Set-ItemProperty -Path $using:Path -Name $using:Name -Value $using:Value
        } @splat
}
{% endhighlight %}


### Using Credential Parameters


Now that you've added a credential parameter, how do you use it? There are two approaches that I'll demonstrate here. One is by using `Get-Credential` and the other
is often referred to as hydrating credentials. The first method which uses `Get-Credential` is no different than using a cmdlet that already has credential support. Below
shows you how to call the Set-RemoteRegistryValue function and specify alternate credentials. By using Get-Credential, you'll be prompted for the user name and password.
At that time you'll type them in and the function will execute. This is great for interactive execution, but what if you want some kind of automation to execute it for you? 

_Tip_

When working with alot of credential objects you should consider something like [BetterCredentials](https://www.powershellgallery.com/packages/BetterCredentials/4.4). Here's a blog
post that walks you through installing it and using it. [Using Credentials in your Profile](https://beaudry.io/articles/2016-08/azure-profile)

{% highlight powershell %}
Set-RemoteRegistryValue -Path 'HKLM:\SOFTWARE\Microsoft\WebManagement\Server' `
-Name EnableRemoteManagement -Value 1 -ComputerName Node1 -Credential (Get-Credential)
{% endhighlight %}


### Hydrating Credential Objects


Say you're using something like Jenkins, TeamCity or Octopus Deploy to execute your automation scripts. How do you use the credential parameter then? You certainly don't want to
have it prompt you. Luckly these tools can safely generate protected variables that can then be used by the functions you write. The process of using that secured variable to
generate a credential object is often called hydration. The process for hydrating credential object is as follows. Obtain the password via a sensitive variable which is delivered
by the automation tool (Jenkins,TeamCity,Octopus Deploy, etc...). Use that password and a username to create a credential object. Then pass that newly created object to the function.
Below is an example of this process. 

{% highlight powershell %}
$Credential= New-Object -TypeName System.Management.Automation.PSCredential `
-ArgumentList $UserName,($Password | ConvertTo-SecureString -AsPlainText -Force)

Set-RemoteRegistryValue -Path 'HKLM:\SOFTWARE\Microsoft\WebManagement\Server' `
-Name EnableRemoteManagement -Value 1 -ComputerName Node1 -Credential $Credential
{% endhighlight %}


### Dehydrating Credential Objects

The last thing I want to cover in the post is how to dehydrate the credential objects. In other words how do we get a username and password out of a credential object? Why would you
want to do this? Well like I said, there are cmdlets, WSDLs, and command line utilites that do not use a credential object which forces us to use usernames and passwords. This isn't an excuse
to use plain text passwords in your code though. You can still use credential objects \ parameters and here's how. Below, I have a function `Bad-Function` that uses `Invoke-Sqlcmd` within it.
`Invoke-Sqlcmd` is one of the cmdlets that still uses `-Username` and `-Password`. To get the credentials from the object we can use the `GetNetworkCredential` method as shown below.

{% highlight powershell %}
function Bad-Function {
    param(
        $Name,
        $Value,
        [ValidateNotNull()]
        [System.Management.Automation.PSCredential]
        [System.Management.Automation.Credential()]
        $Credential = [System.Management.Automation.PSCredential]::Empty          
    )
        
        $UserName = $Credential.GetNetworkCredential().UserName
        $Password = $Credential.GetNetworkCredential().Password   
        
        $splat = @{
            UserName = $UserName
            Password = $Password
            ServerInstance = 'SQL01'
            Query = "SELECT * FROM information_schema.tables WHERE TABLE_TYPE='BASE TABLE'"
        }
        
        Invoke-Sqlcmd @splat
}
{% endhighlight %}

### Sources

https://powershell.org/forums/topic/using-credential-param-as-optional/