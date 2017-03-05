---
layout: post
title:  "Configuring an HTTPS Pull Server for Desired State Configuration PowerShell Version 5"
date:   2017-03-01 09:02:00
comments: true
tags: [PowerShell, DSC, DesiredStateConfiguration, LCM, Pull, PullServer, HTTPS]
modified: 2017-03-01
---
#### Applies to: Windows PowerShell 5.0

#A Complete Guide for Setting up an HTTPS Pull Server for Desired State Configuration PowerShell Version 5

In a previous [blog post](http://duffney.io/Configure-HTTPS-DSC-PullServer) I walked through the setup of an HTTPS pull server, at the time of the writting there was only one way to setup a pull server with HTTPS. Since that blog post was published Microsoft has released another version of the Pull server which I'll refer to as version 2. The offical Microsoft documentation for setting up a pull server can be found [here](https://msdn.microsoft.com/en-us/powershell/dsc/pullserver). There are a few key differences between version 1 and version two. First off, there is no longer a compliance server. Secondly, you now use a registration key to connect to the pull server instead of a configuration ID. A thrid difference is you can now call out which configurations a node should request from the pull server by name. You no longer have to rename the .mof documents on the pull server with the GUI used for the configuration ID. Keep in mind that both methods still work, however the new method does make combining DSC configuration a lot easier.

This blog post will be a complete guide for setting up an HTTPS pull server to deliver DSC configurations. Before you can use a pull server you need an Active Director domain as well as a certificate authority. I'll start this blog post by showing you how to use Lability and DSC to automate the provision of that environment. After the domain is built, I'll walk you through the process of requesting a web server certificate for the pull server. Once you obtained the certificate, I'll show you how to write a DSC configuration for setting up the pull server itself. I then, show you how to publish DSC configurations and DSC resources to the pull server. After that, I show you how to set up the LCM of a client node and requesting a configuration from the pull server. By the end of this blog post you'll have all the knowledge necessary for standing up your own pull secure server for both a lab environment or production environment.


* TOC
{:toc}

## Setting up the AD Domain and Certificate Authority with Lability

### Install Lability
Before you can begin the lab setup you'll need to download the Lability module for the PowerShell gallery. To do that I'll be using a PowerShell v5 cmdlet
called `Install-Module`. Once the module is download be sure to import it into your PowerShell session.


{% highlight powershell %}
Install-Module -Name Lability -Repository PSGallery
Import-Module Lability
{% endhighlight %}


Once the module is installed, it's time to set the default values for the lab folders. The command below sets the locations for the configuration path, hotfix path
and iso path.

### Setup Lab Host Defaults & Directories
{% highlight powershell %}
Set-LabHostDefault -ConfigurationPath C:\Lability\Configurations -HotfixPath C:\Lability\Hotfixes -IsoPath C:\Lability\ISOs
{% endhighlight %}


After the lab default are setup it's time to create the directory structure for Lability. To do that we'll use another cmdlet from the Lability module called
`Start-LabHostConfiguration`. Once the cmdlet runs take a look at the directory structure as shown in the screenshot below. All the directory names are fairly 
self-explanatory, but it's worth familiarizing yourself with the structure.


{% highlight powershell %}
Start-LabHostConfiguration
{% endhighlight %}


![StartLabConfig](/images/posts/2017-02-01\Start-LabHostConfiguration.png "StartLabConfig")


At this point all we've done is setup the directory structure and made sure the Hyper-V feature was turned on. It's now time to download the .iso for the
operating system we'll be using which in this Lab is Windows Server 2016. You can use Windows Server 2012r2 if you want, but you'll want to update PowerShell to v5. In the
course I used Windows Server 2016, so I recommend using that. To download the media issue the following command. _Might take awhile, even with a fast internet connection_

### Install Windows Server 2016 Media
{% highlight powershell %}
Invoke-LabResourceDownload -MediaId 2016_x64_Datacenter_EN_Eval
{% endhighlight %}



### Provision DSC Lab Environment
Now that you have the Lability directories set up and the Windows Server 2016 media download it's time to build the lab environment. In order for Lability to work you'll need two
files [DSCPullServerLab.ps1](https://gist.github.com/Duffney/d62d05b3fd42b4308014bae8c586e184) and [DSCPullServerLab.psd1](https://gist.github.com/Duffney/77f038437abbd742fa3b0614bf6471a4). You can either copy
and paste the code in the GitHub gists and save the code into the two files mentioned or use the following commands to create the two files within your `C:\Lability\Configurations` directory. _DSCPullServerLab.ps1_ is
the DSC configuration Lability will use to automate the setup of the AD domain and certificate authority. _DSCPullServerLab.psd1_ is the configuration data used by the DSC configuration. It also contains some Lability
specific information that provide details on what OS to use for the environment as well as much memory and CPU to give the virtual machine. Once both files are created your configurations directory should look like the
screen shot below.


{% highlight powershell %}
$URI = 'https://gist.githubusercontent.com/Duffney/d62d05b3fd42b4308014bae8c586e184/raw/ec7dad827e7e0a0cd10395d6342e82c0aef2337f/DSCPullServerLab.ps1'
$content = (Invoke-WebRequest -Uri $URI).content
New-Item -Path C:\Lability\Configurations\ -Name DSCPullServerLab.ps1 -Value $content

$URI = 'https://gist.githubusercontent.com/Duffney/77f038437abbd742fa3b0614bf6471a4/raw/e9cb784edad0758bf2244e475e40cfc80fa06cfd/PullServerLab.psd1'
$content = (Invoke-WebRequest -Uri $URI).content
New-Item -Path C:\Lability\Configurations\ -Name DSCPullServerLab.psd1 -Value $content
{% endhighlight %}


![configurationsdir](/images/posts/DSCHTTPSPullServerPSv5/configurationsdir.png "configurationsdir")


With both of these files in place your now ready to run the DSC configuration, but before you can do that you have to load the configuration into memory. To do that I'll dot source _DSCPullServerLab.ps1_ into memory and then
execute it with the configuration data provided by _DSCPullServerLab.psd1._ After the configuration is executed it will generate two .mof documents. One will be called pull.mof and the other will be called pull.meta.mof. If you're
not yet familair with these two files I recommend getting a copy of [The DSC Book](https://leanpub.com/the-dsc-book). It does a great job of breaking down all the componets of DSC.


{% highlight powershell %}
cd c:\Lability\Configurations
.\DSCPullServerLab.ps1
DSCPullServerLab -ConfigurationData .\DSCPullServerLab.psd1 -OutputPath C:\Lability\Configurations\
{% endhighlight %}


![RunLabilityConfig](/images/posts/DSCHTTPSPullServerPSv5/RunLabilityConfig.gif "RunLabilityConfig")


Now that both .mof files are created it now time to start the lab configuration. The cmdlet to do that is `Start-LabConfiguration`. The only mandatory parameter you have to use is `-ConfigurationData`. I however, perfer to add
the `-Verbose` parameter as well, so I can see what Lability is doing. When prompted enter the local administrator password, the same one entered at the time of generating the DSC configurations.


*Delete all external adapters on your Hyper-V host before running this command*


{% highlight powershell %}
Start-LabConfiguration -ConfigurationData .\DSCPullServerLab.psd1 -Verbose
{% endhighlight %}


Once Lability has finished building out the virtual machine, you'll want to power it on. To do that you can go inside the Hyper-V manager and start the vm or
you can use PowerShell to power it up. Below is the command for that. 


{% highlight powershell %}
Start-VM -Name DSC-Pull
{% endhighlight %}


_Coffee break ETA 10 minutes_



## Generate Pull Server Web Certificate


Once the virtual machine has finished booting up it should be configured as both a domain controller and a certificate authority. Log into the virtual machine with the user name of administrator and use the password
you created when generating the .mof documents. Next, open up the PowerShell ISE. We now have to generate the web server certificate that the pull server will use for HTTPS traffic. I'll be using the certutil command line
utility and PowerShell to request the certificate. All the code required to obtain the certificate is listed below. If you'd like to walk through the process in the GUI, follow the steps in my previous [blog post](http://duffney.io/Configure-HTTPS-DSC-PullServer).
You could also use a PowerShell function called [New-DomainSignedCertificate](https://gist.github.com/Duffney/d78b3d9beebcf31aa053256e802ad34f), which I found on Stack. It basically wraps the certutil inside a PowerShell function.

{% highlight powershell %}
$inf = @"
[Version] 
Signature="`$Windows NT`$"

[NewRequest]
Subject = "CN=Pull, OU=IT, O=Globomantics, L=Omaha, S=NE, C=US"
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
FriendlyName = PSDSCPullServerCert
MachineKeySet = TRUE
SMIME = False
PrivateKeyArchive = FALSE
UserProtected = FALSE
UseExistingKeySet = FALSE
ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
ProviderType = 12
RequestType = PKCS10
KeyUsage = 0xa0
"@

$infFile = 'C:\temp\certrq.inf'
$requestFile = 'C:\temp\request.req'
$CertFileOut = 'c:\temp\certfile.cer'

mkdir c:\temp
$inf | Set-Content -Path $infFile

& certreq.exe -new "$infFile" "$requestFile"

& certreq.exe -submit -config Pull.globomantics.com\globomantics-PULL-CA -attrib "CertificateTemplate:WebServer" "$requestFile" "$CertFileOut"

& certreq.exe -accept "$CertFileOut"
{% endhighlight %}


## Writting the Pull Server DSC Config


We now have everything we need to build a pull server. Because DSC is meant to automate the configuration of Windows Servers, I will of course use a DSC configuration to setup the pull server. This configuration has three parameters Nodename,certificateThumbPrint, and RegistrationKey. Nodename will be the name of the pull server. CertificateThumbPrint is the certificate thumbprint of the web server certificate we generated previously. This part can be tricky becasue if you're using certificates to encrypt the mofs there will be a different thumbprints. Just keep in mind this is the thumbprint of the web server certificate which encrypts the HTTPS traffic not the .mof files. The last parameter, RegistrationKey needs a little explaining. In version "2" of the pull server this registration key is used to authenticate the client node with the pull server. It's noting more than a GUID stored in a text file, but it's very important. This replaces the configuration ID used by version "1" of the pull server. This registration key will be used again when we configure the LCM of the client node that connects to the pull server.


The pull server configuration consists of three DSC resources; WindowsFeature, xDscWebService, and File. WindowsFeature and File are built-in DSC resources that are part of the PSDesiredStateConfiguration module. These two are included with every Windows Server that works with DSC. The WindowsFeature resource is being used to install the Windows feature _DSC-Service_, which is required for the pull server to work. The File resource is being used to create a file called _RegistrationKeys.txt_ and to set the contents. The contents of the file is the registration key and as I mentioned previously, this key is just a GUID. _xDscWebService_ is the DSC resource that is setting up the pull server. Since it's a HTTPS "Web" pull server it will be performing some IIS tasks. 


Within the configuration block of _xDscWebService_ there are several propterties worth mentioning. EndpointName is the name of the pull server service. Port is of course the port the pull server will use for it's communications. PhysicalPath is the physical path for the web service. CertificateThumbPrint is the thumbprint of the web server certificate we generated previously, this is populated by the certificateThumbPrint parameter. ModulePath is the path the pull server will store the modules it distributes to the client nodes. ConfigurationPath is the path where the pull server stores the DSC configurations for the client nodes. UseSecurityBestPractices is a boolean value, when it's set to true it enforces the use of stronger encryption cypher. For more information about this setting read the read.me of the [xPSDesiredStateConfiguration](https://github.com/PowerShell/xPSDesiredStateConfiguration). There are a few other properties, but they're self explaintory.


{% gist 1bd49fae18da35c811488326cfb441cb %}


## Deploying the Pull Server Config
    
Since it's the pull server we are configuring we can't have it pull it's own configuration just yet. We'll have to push the configuration you just wrote above to the pull server, which is easily done. I'll be performing this task on the pull server, but you could easily deploy it from a remote authroing machine as well. Before we can start the DSC configuration we'll need two things a GUID and the certificate thumbprint of the web server certificate. Generating a new GUID is easy, just use the newGUID method of the GUID class like this `[guid]::newGuid()`. I always store it inside a variable `$guid` so, if for any reason I can use it again later. Obtaining the certificate thumbprint isn't difficult either. Since we provided a nice and freindly name of _PSDSCPullServerCert_ it's easily found with `Get-ChildItem`. The path to the certificate is `Cert:\LocalMachine\My`, I'll use Where-Object to filter the results `Get-ChildItem Cert:\LocalMachine\My | where {$_.FriendlyName -eq 'PSDSCPullServerCert'}`. Again I tyipcally store this in a variable so I can resue it later if needed. I normally name that variable `$cert`.


{% highlight powershell %}
$guid = [guid]::newGuid()

$cert = Get-ChildItem Cert:\LocalMachine\My | where {$_.FriendlyName -eq 'PSDSCPullServerCert'}
{% endhighlight %}


Now that we have both the guid and the certificate information we can run the pull server dsc configuration which will generate the pull server .mof document! Remember to load the configuration into memory before attempting to execute the configurtion. Once the configuration is loaded into memory you can execute it by specifiying the name and then any paramters it had. I've named this configuration _DscPullServer_, you'll have to provide all the paramters as I've done below. The value for the certificateThumbPrint is $cert.Thumbprint because the thumbPrint is a property of the $cert variable object. -RegistrationKey is the guid we generated and I'm also providing a specific output path for the mof file once it's generated. 


{% highlight powershell %}
DscPullServer -certificateThumbPrint $cert.Thumbprint -RegistrationKey $guid -OutputPath c:\dsc
{% endhighlight %}


With the mof document generated the last thing we need to do before you have an operational pull server is to push the configuration. The cmdlet for that is `Start-DscConfiguration`, the only mandatory parameter is -Path. Which is used to provide the path to the mof document you just generated. I also recommend using -Wait and -Verbose, so you can see what the configuration is doing. By default the cmdlet creates a background job to run the configuration in, using the -Wait pushes the output to the console. -Verbose displays the pretty blue output text. 


{% highlight powershell %}
Start-DscConfiguration -Path C:\dsc -Wait -Verbose
{% endhighlight %}


![pullconfigverbose](/images/posts/DSCHTTPSPullServerPSv5/pullconfigverbose.png "pullconfigverbose")

## Testing the Pull Server


Before we move on, it's a good idea to confirm that the pull server is operating correct. To test the pull server open up Internet Explorer and to go ` https://pull:8080/PSDSCPullServer.svc`. You should get some xml results returned to you. If you do, congradulations! You have a pull server! _if you changed the name of the pull server, you'll have to update this URL_. If you want to do it the *PowerShell* way, you can use `Invoke-WebRequest` as well. The pull server works nicely with server core, so learing the PowerShell way is always a good choice. When using the Invoke-WebRequest method, make sure the StatusCode returns 200 and the content contains some xmls stuff.

{% highlight powershell %}
Invoke-WebRequest -Uri 'https://pull:8080/PSDSCPullServer.svc/' -UseBasicParsing
{% endhighlight %}

![verifypullserver](/images/posts/DSCHTTPSPullServerPSv5/verifypullserver.png "verifypullserver")


## write dsc config for pull server client
    explain Configuration names replaced node name specific configs

## publish dsc resources and config to the pull server 

## write LCM Configuration

## push lcm config

## update dsc Configuration