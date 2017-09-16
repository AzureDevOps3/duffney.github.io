---
layout: post
title:  "Getting Started with Invoke-Build Part 1"
date:   2017-09-15 09:02:00
comments: true
tags: [PowerShell, Modules, ContinuousIntegration, CI, Continuous, Integration, Invoke-Build, InvokeBuild]
modified: 2017-09-15
---
### Table of Contents
* TOC
{:toc}

### What is InvokeBuild?

Invoke-Build is a build automation tool written in PowerShell. What does that mean? Well, just like anything written in PowerShell the purpose of it is to automate something and in this case it's to automate the building of software artifacts. What does building software have to do with PowerShell development? Well, if you've gotten into building your own PowerShell modules and publishing them to a file share, internal feed, or PSGallery you'll see the value in this tool. Before we dive into how to use Invoke-Build, let's take a look at what you might automate with Invoke-Build.

As I mentioned Invoke-Build is used to automate the building of artifacts for software. In the case of PowerShell development that artifact means a .zip file or .nuget file containing a PowerShell module. Invoke-Build is used to automate the creation of that artifact and the publishing of that artifact to some destination be it the PSGallery or an internal feed. Before you can automate something you have to understand the manual process, so let's take a look at a typical workflow for creating a PowerShell module artifact.

### PowerShell Module Development Workflow

PowerShell module development by no means has a _standard process_ with that said I'll explain my workflow as it relates to the building of a module artifact with the intent of distributing the module to other systems besides my client computer. This example workflow starts after you have created or made a change to a PowerShell module and you now wish to package it up and distribute it. I won't be covering PowerShell module design patterns in this post. The process I follow for creating a module artifact goes something like this; install dependencies, analyze code with a linting tool `PSScriptAnalyzer`, execute Pester tests, confirm tests passed, update the module manifest with new functions and increase module version, generate an artifact, and publish the artifact.

*Workflow*
1. Install Dependencies
1. Analyze Code
2. Test Code
3. Confirm Tests Passed
4. Update Module Manifest
5. Archive & Publish New Artifact