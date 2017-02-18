---
layout: post
title:  "Adding Credentials to PowerShell Functions"
date:   2017-01-29 09:02:00
comments: true
tags: [PowerShell, Credentials, PSCredentials]
modified: 2017-01-29
---

In this blog post, I'll show you how to add credential parameters to PowerShell functions. But, before I do that let's first talk about why you'd want to add a credential parameter to your functions.
The purpose of the credential parameter is to allow you to run the function and or cmldet as a different user. Some account other than the one currently running the PowerShell session. The most common use is to run
the function or cmdlet as an elevated user account. For example the cmdlet `New-ADUser` has a `-Credential` parameter, which you could provide domain admin credentials to in order to create an account in a domain. Assuming
your normal account running the PowerShell session doesn't have that access already. This blog post walks you through the process of adding such functionality to your PowerShell functions. I also discuss how to get around
common issues when working with _legacy_ cmdlets that don't support a credential object, but before we get started let's first talk about PSCredential objects and how to generate them.


## Creating Credential Object


_PSCredential objects represent a set of security credentials, such as a user name and password._ [MSDN](https://msdn.microsoft.com/en-us/library/system.management.automation.pscredential(v=vs.85).aspx) The objects
are then passed to the parameter of a function and used execute the function as that user account in the credential object. There are a few ways that you can generate a credential object. The first and easiest method
is by using the PowerShell cmdlet `Get-Credential`. You can simply execute `Get-Credential`, which will result in a username and password prompt. From there you could enter the _domainName\userName_ or you can call the
cmdlet with some optional parameters. To specify the domain name and user name ahead of time you can use either the `-Credential` or `-UserName` parameters. The only difference I've noticed is when you use `-UserName` you'll
also be required to input a message value. The code below demonstrates using the cmdlet. You can also store the credential object in a variable, which allows you to use the credential several times. In the below example
I'm storing each credential object to a variable called $Cred.


{% highlight powershell %}

$Cred = Get-Credential

$Cred = Get-Credential -Credential domain\user

$Cred = Get-Credential -UserName domain\user -Message 'Enter Password'

{% endhighlight %}


Sometimes, you won't want an interactive method of creating credential objects as I just demonstrated. Most automation tools such as Jenkins, TeamCity and Octopus Deploy require a non-interactive method. To do this you'll have
to create a secure string, which contains the password. You then have to pass the secure string and user name to the `System.Management.Automation`'s PSCredential method. Sounds a lot more complicated than it is I assure you. The
syntax for creating a secure string looks like this `ConvertTo-SecureString “PlainTextPassword” -AsPlainText -Force`. Both the `-AsPlainText` and `-Force` parameters are required or you'll receive error messages saying you shouldn't
pass plain text into a secure string. Reason being, if your PowerShell session is logged that password would exist in the log. With the secure string created you'll need to pass it to the PSCredential method to create the credential
object. That syntax loos like this `New-Object System.Management.Automation.PSCredential (“username”, $secpasswd)`. In the example below, I'm storing the secure string into a variable called $password and the credential object
into a variable $Cred. 

{% highlight powershell %}

$password = ConvertTo-SecureString “PlainTextPassword” -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential (“username”, $password)

{% endhighlight %}


Now, that you know how to create credential objects, it's now time to talk about how we add credential parameters to our PowerShell functions. 

## Adding a Credential Parameter


Just like any other parameter, you start off by adding it in the param block of your function. I typically use the parameter name of $Credential because that's what existing PowerShell cmdlets use.
With the parameter added you then have to define it's type which you just learned is `[System.Management.Automation.PSCredential]`. Before we move on take a moment to look at the code snippet below.
It is for a function called Get-Something. It has two parameters $Name and $Credential which has the `[System.Management.Automation.PSCredential]` type. 

{% highlight powershell %}
function Get-Something {
    param(
        $Name,
        [System.Management.Automation.PSCredential]$Credential      
    )
{% endhighlight %}

The above code would be enough to have a working credential parameter, however there are a few things you can add to make it more robust. The first thing you can add is `[ValidateNotNull()]`, which checks to see
if the value being passed to `-Credential` is null. If it is, it will stop the function from executing. If you don't have a proper credential object and specified the `-Credential` parameter why execute? The next
thing you can add is `[System.Management.Automation.Credential()]`. This allows you to pass in a username as a string and have an interactive prompt for the password, which I'll demonstrate later in the post. The last,
thing you can do is set a default value for the $Credential parameter. Adding `[System.Management.Automation.PSCredential]::Empty` as a default value will populate an empty credential object. Why do this? Well, in your
code you might be passing this $Credential object to existing PowerShell cmdlets that uses the `-Credential` parameter. If you do not provide a credential object to your function the code will error out when it hits the cmdlet 
inside your code that requires a credential. By providing a default empty credential object you can resolve that error. There are a few other methods for handling this problem. One, is a simple if statement and another option is
to use splatting. I'll walk through this later in the post. The below snippet of code, shows what the function would now look like with all these changes.

_Tip_

"Caveat: Some cmdlets that accept a Credential parameter do not support/check for [System.Management.Automation.PSCredential]::Empty like they should. This should be treated as a bug by those cmdlet authors.
By using an if statement or by using splatting, we can get around this limitation. See the Dealing with Legacy Cmdlets section.


{% highlight powershell %}
function Get-Something {
    param(
        $Name,
        [ValidateNotNull()]
        [System.Management.Automation.PSCredential]
        [System.Management.Automation.Credential()]
        $Credential = [System.Management.Automation.PSCredential]::Empty  
    )
{% endhighlight %}


## Using Credential Parameters

In this next section of the post, I'm going to demonstrate how to use credential parameters. I'll be using a function called `Set-RemoteRegistryValue`, which is out of [The Pester Book](https://leanpub.com/the-pester-book).
I've added the credential parameter by using the technqiues you just learned above. Inside the `Set-RemoteRegistryValue` function it uses Invoke-Command. I've updated it to use the `-Credential` parameter and then added the
$Credential variable created by the function. This allows me to change the user who's running Invoke-Command. Because I've included `[System.Management.Automation.PSCredential]::Empty` as the default value, it should work with
no credentials being passed and with credentials being passed. So let' test it out and see if it works as we expect. The `Set-RemoteRegistryValue` code is below if you'd like to follow along.

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


### With Credentials

The first way I can use the -Credential parameter of `Set-RemoteRegistryValue` is by using `Get-Credential` in `()` at run time. This will cause the `Get-credential` to run first just like a math problem. 
You'll then be prompted for a user name and password. You could use the `-Credential` or `-Username` parameters of `Get-credential` to pre-populate the user name and domain. If you're not familair with a
technqiue called splatting, that's how I'm passing the rest of the parameter to the `Set-RemoteRegistryValue` function. For more information about splatting, check out this [MSDN article](https://msdn.microsoft.com/en-us/powershell/reference/5.0/microsoft.powershell.core/about/about_splatting)

{% highlight powershell %}
$remoteKeyParams = @{
ComputerName = $env:COMPUTERNAME
Path = 'HKLM:\SOFTWARE\Microsoft\WebManagement\Server'
Name = 'EnableRemoteManagement'
Value = '1' 
}

Set-RemoteRegistryValue @remoteKeyParams -Credential (Get-Credential)
{% endhighlight %}

![GetCredAtRunTime](/images/posts/AddingCredsToFunctions/GetCredAtRunTime.gif "GetCredAtRunTime")

Having to use `(Get-Credential)` seems a little weird doesn't it? Normally, when you run cmdlets that support the `-Credential` parameter you can just put in your username and it will automatically prompt
for the password. Well, because we used `[System.Management.Automation.Credential()]` in the function we can do that! Just remember to put the user name as a string right after the `-Credential` parameter
and you'll get prompted for your password.

{% highlight powershell %}
$remoteKeyParams = @{
ComputerName = $env:COMPUTERNAME
Path = 'HKLM:\SOFTWARE\Microsoft\WebManagement\Server'
Name = 'EnableRemoteManagement'
Value = '1' 
}

Set-RemoteRegistryValue @remoteKeyParams -Credential duffney
{% endhighlight %}

![GetCredsPrompt](/images/posts/AddingCredsToFunctions/GetCredsPrompt.gif "GetCredsPrompt")

_Tip_

If you're following allong you'll need to install a few windows features to create this registry value.

`Install-WindowsFeature Web-Server`
`Install-WindowsFeature web-mgmt-tools`

### With Credentials in a variable

You can also populate a credential variable ahead of time and pass it to the `-Credential` parameter of `Set-RemoteRegistryValue` function. I use the follwoing method a lot when working with
continious integration and continious deployment tools such as Jenkins, TeamCity and Octopus Deploy. I'll use the .Net method to create the credential object as well as use a secure string
to pass in the password. You'll notice the password is in clear text when I create the secure string. All of the CI and CD tools I just mentioned have a secure method of populating that at run time
so when using those tools I replace the plain text password with a variable defined within the tool I'm using. For an example of that check out Hodge's blog post [Automating with Jenkins and PowerShell on Windows - Part 2](https://hodgkins.io/automating-with-jenkins-and-powershell-on-windows-part-2).

{% highlight powershell %}
$password = ConvertTo-SecureString “P@ssw0rd” -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential (“duffney”, $password)

$remoteKeyParams = @{
ComputerName = $env:COMPUTERNAME
Path = 'HKLM:\SOFTWARE\Microsoft\WebManagement\Server'
Name = 'EnableRemoteManagement'
Value = '1' 
}


Set-RemoteRegistryValue @remoteKeyParams -Credential $Cred
{% endhighlight %}



### Without Credentials

Since I added `[System.Management.Automation.PSCredential]::Empty` as the default value of the `-Credential` parameter, I can run the command without credentials as well. Remember, that not all
cmdlets that have the `-Credential` parameter allow for this. Not to worry though, we can get around this limitation and I'll discuss how in the next section of the post.

{% highlight powershell %}
$remoteKeyParams = @{
ComputerName = $env:COMPUTERNAME
Path = 'HKLM:\SOFTWARE\Microsoft\WebManagement\Server'
Name = 'EnableRemoteManagement'
Value = '1' 
}

Set-RemoteRegistryValue @remoteKeyParams
{% endhighlight %}


## Dealing with Legacy Cmdlets

Being in the Tech industry, you'll never escape the need to support and or deal with legacy applications. Working in PowerShell is no different and in this case you'll eventually run into one or both
of the following problems. A cmdlet doesn't support `[System.Management.Automation.PSCredential]::Empty`, which I've mentioned a few times. Or, the cmdlet you want to use doesn't even support the `-Credential`
parameter at all and instead accepts a string username and string password! This section of the blog post is dedicated to helping you solve this problems. First up is, what to do when a cmdlet doesn't
support `[System.Management.Automation.PSCredential]::Empty`.

Before I dive into solving this problem, let me first expand on what the problem is. So, what does it mean when I say the cmdlet doesn't support `[System.Management.Automation.PSCredential]::Empty`?
it means that when you do not provide a credential object the function you wrote will fail because the cmdlet inside your function will fail as it's unable to accept the empty credential object. The
function you wrote will work wonderfully as long as you supply a credential object, but as soon as you don't no dice. So, how do we fix it? There are a few ways, the first is to check the `-Credential`
parameter for a value with an if else statement. The if statement checks the value of the $credential and adds the `-Credential` parameter to `Invoke-Command` only if it's not empty, otherwise
it issues the `Invoke-Command` without the `-Credential` parameter.

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

    if($Credential -ne [System.Management.Automation.PSCredential]::Empty) {
        Invoke-Command -ComputerName:$ComputerName -Credential:$Credential  {
            Set-ItemProperty -Path $using:Path -Name $using:Name -Value $using:Value
        }
    } else {
        Invoke-Command -ComputerName:$ComputerName {
            Set-ItemProperty -Path $using:Path -Name $using:Name -Value $using:Value
        }
    }
}
{% endhighlight %}

Another way to address this problem is to use splatting. I still use an if statement to determine if $credential is empty or not, but the difference is I'm just adding a $credential object
to a hashtable, instead of repeating the entire block of code that uses `Invoke-Command`. To learn more about splatting inside functions which out a previous blog post of mine [Splatting Parameters Inside Advanced Functions](http://duffney.io/Splatting-Parameters-Within-AdvancedFunctions). That wraps up dealing with cmdlets that don't support `[System.Management.Automation.PSCredential]::Empty`,so lets move on to dealing with cmdlets that don't even have a `-Credential` parameter.

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


A good example of a cmdlet that accepts a string as a password is `Register-ScheduledTask`. If you're a Windows admin, you're very familair with scheduled task. They're extremely useful, but do 
have some limitations. Like, requiring a plan text password... Don't worry, you can get around this and avoid storing passowrd within your PowerShell scripts. To get around this I'll show you how to
hydrate and dehydrate a credential object. Which is fancy talk for create a credential object and extract out the user name and password from it. 


### Adding a Credential Parameter -OLD


Adding a credential parameter is easy enough. Instead of using a Username and Password params you just add a single parameter called Credential. The
name of the parameter can be anything you want, but since most PowerShell cmdlet that support credential objects use that name I tend to stick with it.
Below, I'm using a function called Set-RemoteRegistryValue which is out of [The Pester Book](https://leanpub.com/the-pester-book). I've added the credential parameter
within the param block and also added the _-Credential_ parameter to the `Invoke-Command` within the function. This will allow me to run this function with alternate
credentials. By using a default parameter value of `[System.Management.Automation.PSCredential]::Empty` I can prevent the _-Credential_ param from erroring out when the parameter isn't used. Which means the
function will run when a credential is specified and when it is not. This is one of serveral ways to handle credential parameters. Another would be to use splatting. If you're
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
https://blogs.msdn.microsoft.com/koteshb/2010/02/12/powershell-how-to-create-a-pscredential-object/


### furture learnings

cred management