---
layout: post
title:  "Creating an AWS EC2 Windows Instance with Cloudformation and PowerShell"
date:   2019-08-10 13:37:00
comments: true
modified: 2019-08-10
---

In this blog post I'll be walking through how to use the [AWSPowerShell.NetCore](https://www.powershellgallery.com/packages/AWSPowerShell.NetCore/3.3.365.0) PowerShell module to deploy a Windows virtual machine to AWS EC2. If you're unfamilair with AWS EC2 stands for Amazon Web Services Elastic Compute Cloud. It's AWS IaaS offering that allows you to spin up virtual machines in their cloud. Cloudformation is another service offered by AWS that allows you to define your infrastructure as code (IaC). And PowerShell well... it says it all. It is a powerful shell. 


* TOC
{:toc}


### Creating a Key Pair

In order to provision a useful Ec2 instance, I'll first need to create a key pair. AWS Ec2 uses public–key cryptography to encrypt and decrypt login information. As you'll see later, I'll use this key to decrypt the password for the Windows instance in Ec2. Luckily, I can do this in PowerShell. The cmdlet to create new AWS Key Pairs is `New-EC2KeyPair`. The results of that cmdlet has a member called `KeyMaterial` that contains the private key for the keypair. Which I'll need to save as a .pem file. If I do not do that I won't be able to decrypt the Ec2 instances administrator password to login. Note before you can run the following commands you'll need the AWS PowerShell module for your operating system installed and setup to connect to your AWS account. Check out [Guide: Setting up the AWS PowerShell Module]("https://www.youtube.com/watch?v=Z4rNHjEXoSs") to learn more.


```
$awsPSKeyPair = New-EC2KeyPair -KeyName awsPSKeyPair
$awsPSKeyPair.KeyMaterial | Out-File -Encoding ascii awsPSKeyPair.pem
```

To confirm the key pair was generated and is availabe in AWS we can run the cmdlet `Get-Ec2KeyPair`

```
Get-EC2KeyPair -KeyName awsPSKeyPair
```

### Create the Cloudformation Template

Now that a key pair exists, I can move on to creating a Cloudformation template. A Cloudformation template is a document that defines the AWS resources you want to provision. Cloudformation calls the combination of those resources a _stack_. I've used several infrastructure as code (IaC) and Cloudformation is by far one of my favorites. Cloudformation allows you to write the IaC documents in one of two formats; JSON or YAML. I write a fair amount of ansible code at my day job so I'm going to choose to write it in YAML. 

In order to get this Ec2 instance and accessible to me I'll need a few things. First off, I'll need to assign the Ec2 instance a key pair so I can use the .pem file to decrypt the password. Secondly, I'll need the Ec2 instance itself. I want it to be a Windows 2019 virtual machine. AWS offers prebuilt templates called AMIs (Amazon Machine Image). I'll also need to decide on an instance type (how big I want the virtual machine). I want to stay in the free-tier of AWS so I'll stick with t2.micro. For more information on instance types check out the [AWS Instnace Types docs](https://aws.amazon.com/ec2/instance-types/). Lastly, I'll also need a way to access this Ec2 instnace via RDP. To do that I'll need to create a security group and allow RDP access on port 5985. I also added some bootstrapping command under the userdata section that installs [chocolatey](https://chocolatey.org/).

There is a lot to learn when it comes to Cloudformation and I myself am just scratching the surface. An excellent way to get a great foundation understanding of Cloudformation is to check out acloudgurus's [Introduction to AWS CloudFormation](https://acloud.guru/learn/intro-aws-cloudformation).

{% gist 3e9dae330f9f76ea65a744cfdfe43d7a %}

### Writing Cloudformation Templates to AWS S3 with PowerShell

At this point we have a key pair and a Cloudformation template. Where should I put this template file? Of course, I could create a new repository on Github, but I want to learn more about AWS. Another option is to store in AWS S3 (Simple Storage Service). AWS S3 has a concpet called buckets. You can think of buckets as network shares. In the bucket you can create directories and or upload files. Before you can store things in S3 you'll need a bucket. The bucket name does need to be unqiue across off of AWS and in lower case. Once the bucket is crated you can start to upload files to it. 

```
#Create new bucket
New-S3Bucket -BucketName duff-awspsbucket
```

### Deploying a Cloudformation Stack with PowerShell