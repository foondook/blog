---
layout: post
title: "Continuous Delivery made easy"
teaser: "Personally, I always prefer the easy way, or the way with the most fun. Maybe this is the reason why I became a professional software developer many years ago. (Sixteen years actually!) Sometimes you need to have some fun with cool tools, to make some tasks as easy as possible. At the end it is both, fun at work and making things easy. Seriously, I'm really going to write about Continuous Delivery :-)"
author: "Jürgen Gutsch"
comments: true
image: /img/cardlogo-dark.png
tags: 
- .NET Core
- Unit Test
- XUnit
- MSTest
---

Personally, I always prefer the easy way, or the way with the most fun. Maybe this is the reason why I became a professional software developer many years ago. (Sixteen years actually!) Sometimes you need to have some fun with cool tools, to make some tasks as easy as possible. At the end it is both, fun at work and making things easy. 

Seriously, I'm really going to write about Continuous Delivery :-)

Some developers really hate it. I also did in the past, but I did the whole thing wrong or I used the wrong tools. I did CD when the Software was almost finished, or pretty much too complex to create a simple CD process. 

## When should I start to set-up the CD process?

Start to setup the CD process as early as possible. As best, you should start setting up a CD before you start writing any line of code. You should know the rough architecture. You should know how the app will be used and how it will run on the target machine. Is it a web app, a mobile app or a desktop app? 

The first thing I do, when I start a new project, is to set-up the CD. Except in personal projects, that wasn't started as a real project initially (e.g. the GraphQL Middleware and the GraphiQL Middleware)

## What are the parts of CD I should set-up?

In the most cases you should have almost the same five steps to do:

* Preparing
* Building
* Testing
* Packaging
* Deploying

## What if I don't yet know all the relevant aspects?

All of this steps will grow over time and you don't need to be complete after the Initial setup. There will be many unknown aspects, you cannot know from the beginning. So start with the known ones. It is important to start anyway. It is absolutely fine to add or change some some things later on. This changes and improvements will be done with pretty much less afford, then.

## How does the initial set-up look like?

Some of this steps will have sup-steps. E. g. the preparing step, could be divided into 

* Clean Workspace
  * e. g. remove old artifacts
* Restore dependencies
  * e. g. NuGet restore
* Pre-Configure the build
  * e. g. set the assembly versions, etc.

The testing part could be about unit-testing, integration-testing, UI-testing, etc. and a lot more.

With this the first simple initial setup could look like this.

* Preparing
  * Clean-Workspace
  * NuGet restore
  * Change AssemblyInfos
* Build the solution
* Run all tests in *.tests.dll 
* Publish the application
* zip the output and move to a share

## Where should the CD run?

When you create a CD process, you should build it in a way that it is able to run everywhere. Don't create the process for a specific tool like TFS, Jenkins, TeamCity, or whatever. Every developer in the team who checks out the code, should run the CD on his developer machine. For sure, it should even run on the build machines.

This ensures, that every developer is able to maintain the process, and that every developer can try and debug things in the CD process, before committing it to the server. This makes CD development a lot faster, than configuring every single step on the build servers UI, blocking the servers and other developers and waiting until it is finished. 

## How does it look like to get it running everywhere?

Scripting is the magic here, to get it running on every machine. Use whatever script language you like. Batch, Shell, PowerShell, Python, NodeJS. Or use specialized scripting engines like Grunt, Gulp, WebPack, MAKE, RAKE, FAKE or CAKE. 

In the past I used FAKE a lot, but this couldn't get excepted by the most C# devs in the Team, because it is build on top of F#. It seems they are in fear of C#, even if they don't need to learn F# to create a FAKE build. I know it is not fear, but respect. That's why I switched to CAKE, which is build on top of C# Script. 

If you have a script done, you can start it locally on your machine, or you can start it on the build server. You can start it on every machine where the relevant scripting runtime is available.

## Does scripting limit you in any case?

I'm pretty sure the opposite is the case. There are pretty much more possibilities, if you use a scripting language.

Scripting is much more faster in development and deployment:

* no compiling is needed
* changes take affect immediately
* any text editor can be used

CAKE or FAKE have a lot of features built-in to support many different tasks:

* File system handling
* Source code management 
* IDE tasks
* Test tools
* Starting external processes
* and many more

Only the execution performance is slower compared with a compiled program, but this is not really relevant, because the most time is needed by the execution of the build, the test executions and the deployments. Some milliseconds more or less on the build runtime doesn't really matter here...

The other good things about FAKE or CAKE are, because it is build on top of a runtime that supports .NET DLLs, you are able to develop and to use custom and compiled code  written in C#. If you miss a feature and cannot find it in the community, built it yourself and (hopefully) contribute it to the community. I didn't need to do it until now, because all features I needed, were already in. 

## Are there any things scripts can't do?

Sure. There are also some aspects scripts are not responsible for. All the things around the execution of the following tasks, should be done on a build server:

* Scheduling / triggering the build
  * scheduled on a regular base 
  * or triggered by an external system, e.g. the SCM
* Fetching the sources from the SCM
  * because the build script is part of it
* Executing the build script
* Analyzing the build log
  * search for bad return coded
  * specific keywords, etc.
* Report the build state 
  * build time
  * build and test quality
* notify the relevant persons
  * at least notify about quality issues

These are also the tasks that don't need to be done on the developer machine. And FAKE or CAKE do a nice output and a nice summery too.

> I had a nice experience with an early version of Jenkins, which parses the output for the standalone keyword "error" or "failed". I got a broken build but a successful execution of all build tasks. The build was good, the tests were successful, even the deployment ends without any error. But one single test wrote that keyword to the console. And Jenkins set the complete build to the failing state because of that. This is fixed now. 

## Ho do I install and execute CAKE?

Installing a CAKE script is pretty easy. Open a PowerShell and CD to the solution folder or any other folder where you want to run the CAKE scripts. Execute this line in the PowerShell console:

~~~ powershell
Invoke-WebRequest https://cakebuild.net/download/bootstrapper/windows -OutFile build.ps1
~~~

This just downloads the latest build.ps1 to that folder. This file is the bootstrapper for CAKE. It checks the environment, installs CAKE if needed and executes the build script.

Add a new file called build.cake, add that contend and safe the file

~~~ csharp
var target = Argument("target", "Default");

Task("Default")
    .Does(() =>
    {
      Information("Hello World!");
    });

RunTarget(target);
~~~

This is C# Script executed on Roslyn. CAKE will automatically add the most important dependencies. There is also a preprocessor running, which installs dependencies on demand.

Run that script in the PowerShell console by just calling the build.ps1

~~~ powershell
.\build.ps1
~~~

That is the first execution of a "Hello World!" build script. The first call takes a little more time, because it installs all the needed stuff. After that it compiles and executes the CAKE script, Task, by Task.

## How does a CAKE script look like?

The first Hello World example shows you a little bit. But let's have a look into a complete build, test and deployment script for a ASP.NET Core application.

~~~ csharp
#addin "nuget:https://api.nuget.org/v3/index.json?package=Cake.WebDeploy"
var target = Argument("target", "Default");

var siteName = Argument("site-name", "cake-demo");
var deployPassword = EnvironmentVariable("DEPLOY_PASSWORD");
var publishDir = "./Published/";

Task("Clean")
	.Does(() => 
	{
		CleanDirectory(publishDir);
		DotNetCoreClean("./aspnetcore-cake.sln");
	});

Task("Restore")
	.IsDependentOn("Clean")
	.Does(() => 
	{
		DotNetCoreRestore("./aspnetcore-cake.sln");
	});

Task("Build")
	.IsDependentOn("Restore")
	.Does(() => 
	{
		DotNetCoreBuild("./aspnetcore-cake.sln");
	});

Task("Test")
	.IsDependentOn("Build")
    .Does(() =>
	{
		var projectFiles = GetFiles("./*.Tests/*.csproj");
		foreach(var file in projectFiles)
		{
			DotNetCoreTest(file.FullPath);
		}
	});

Task("Publish")
	.IsDependentOn("Test")
	.Does(() => 
	{
		var settings = new DotNetCorePublishSettings
		{
			OutputDirectory = publishDir
		};
		DotNetCorePublish("./ProjectToBuild/ProjectToBuild.csproj", settings);
	});

Task("Deploy")
	.IsDependentOn("Publish")
    .Does(() =>
    {
        DeployWebsite(new DeploySettings()
        {
            SourcePath = publishDir,
            SiteName = siteName,
            ComputerName = "https://" + siteName + ".scm.azurewebsites.net:443/msdeploy.axd?site=" + siteName,
            Username = "$" + siteName,
            Password = deployPassword
        });
    });

Task("Default")
	.IsDependentOn("Deploy")
	.Does(() =>
	{
	  Information("Congratulations, your build is done!");
	});

RunTarget(target);
~~~

The first line is a pre-processor directive to install Cake.WebDeploy from NuGet. This is an Add-In needed in the second last Task to publish the application to Microsoft Azure.





