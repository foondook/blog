---
layout: post
title: "Continuous Delivery made easy"
teaser: "description"
author: "Jürgen Gutsch"
comments: true
image: /img/cardlogo-dark.png
tags: 
- .NET Core
- Unit Test
- XUnit
- MSTest
---

Personally, I always prefer the easy way, or the way with the most fun. Maybe this is the reason why I became a professional software developer many years ago. (Sixteen years actually.) Sometimes you need to have some fun with cool tools, to make some tasks as easy as possible. At the end it is both, fun at work and making things easy. 

Seriously, I'm really writing about Continuous Delivery :-)

Some developers really hate it. I also did in the past, but I did the whole thing wrong or I used the wrong tools. I did CD when the Software was almost finished, or pretty much too complex to create a simple CD process. 

## When should I start to set-up the CD process?

As early as possible. As best, you should start setting up a CD before you start writing any line of code. You should know the rough architecture. You should know how the app will be used and how it will run on the target machine. Is it a web app, a mobile app or a desktop app? 

The first thing I do, when I start a new project, is to set-up the CD. Except in personal projects, that wasn't started as a real project initially (e.g. the GraphQL and the GraphiQl Middleware)

## What should I set-up?

In the most cases you should have almost the same five steps to do:

* Preparing
* Building
* Testing
* Packaging
* Deploying

## What if I don't know all the relevant aspects?

All of this steps will grow over time and you don't need to be complete after the Initial setup. There will be many unknown aspects, you cannot know from the beginning. So start with the known ones.

## How does the initial set-up look like?

Some of this steps will have sup-steps. E. g. the preparing step, could be divided into 

* Clean Workspace
  * Remove old artifacts
* Restore dependencies
  * e. g. NuGet restore
* Pre-Configure the build
  * e. g. Set the assembly versions, etc.

The testing part could be about unit-testing, integration-testing, UI-testing, etc. and a lot more.

The first simple initial setup could look like this.

* Preparing
  * Clean-Workspace
  * NuGet restore
  * Change AssemblyInfos
* Build the solution
* Run all tests in *.tests.dll 
* Publish the application
* zip the output and move to a share

## Where should the CD run?

When you create a CD process, you should build it in a way that it can run everywhere. Don't create the process for a specific tool like TFS, Jenkins, TeamCity, or whatever. Every developer in the team who checks out the code, should run the CD on his developer machine. For sure, it should even run on the build machines.

This ensures, that every developer is able to maintain the process, and that every developer can try and debug things in the CD process, before committing it to the server. This makes CD development a lot faster, than configuring every single step on the build servers UI, blocking the servers and other developers and waiting until it is finished. 

## How does it look like to get it running everywhere?

Scripting is the magic. Use whatever script language you like. Batch, Shell, PowerShell, Python, NodeJS. Or use specialized scripting engines like MAKE, RAKE, FAKE or CAKE. 

In the past I used FAKE a lot, but this couldn't get excepted by the most C# devs in the Team because it is build on top of F#. It seems they are in fear of C#, even if they don't need to learn F# to create a FAKE build. I know it is not fear, but respect. That's why I switched to CAKE, which is build on top of C# Script. 

If you have a script completed you can start it locally on your machine, or you can start it on the build server. You can start it on every machine where the relevant scripting runtime is available.

## Does scripting limit you in any case?

I think the opposite is the case. There are pretty much more possibilities if you use a scripting language.

Scripting is much more faster in development and deployment:

* no compiling is needed
* changes take affect immediately
* any text editor can be used

CAKE or FAKE have a lot of tools build-in to support many different tasks:

* File system handling
* Source code management 
* IDE tasks
* Test tools
* Starting external processes
* and many more

Only the execution performance is slower compared with a compiled program, but this is not really relevant, because the most time is needed by the execution of the build, the test executions and the deployments. Some milliseconds more or less on the build runtime doesn't really matter here...

The other good things about FAKE or CAKE are, because it is build on top of runtime that supports .NET DLLs, you are able to develop and to use custom and compiled code  written in C#. If you miss a feature and cannot find it in the community, built it yourself and (hopefully) contribute it to the community. I didn't need to do it until now. 

