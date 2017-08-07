---
layout: page
title: I'm coding...
author: Jürgen Gutsch
comments: false
---

Some of my blog posts are about open source projects I work on. Some of ththis projects em are managed by me. On this page you'll find a list of current (open source) projects i work on:

## LightCore

LightCore is a lightweight and fast dependency injection container initially written by the former Microsoft ASP.NET MVP Peter Bucher. I took over the project and I'm now managing and maintaining that project. And I'm pretty happy to work together with Peter on that project :)

The current work is to create a new version based on .NET Standard 2.0 to get it compatible to the current platforms like .NET Core, Mono, Xamarin and even the .NET Framework

This library is based on .NET Standard 2.0 preview 2

## GraphQL MiddleWare

The is a pretty small library just to get the GraphQl running as middleware in ASP.NET Core projects. This project based on the GraphQL-dotnet library.

This library is based on .NET Standard 1.4 yet and will be updated to .NET Standard 2.0. 

## CQS Library

This is a library that halps you to implement the Command & Query Segregation pattern. The library contains all the important infrastructure code which is the same in every CQS project. You'll get a CommandDispatcher, a Query Processor and the needed interfaces for your Queries and Commands. There's also an option to implement pre-checks for every single command.

This library is based on .NET Standard 1.6 and will be updated to .NET Standard 2.0.

## Simple Object Store

This library was created to have a simple, lightweight data storage for unit-test, demo projects, click dummies and project build with UI driven design. The library isn't really thread save ad it is not recommended to use it in production, even if I run it in a production project that only reads data out of that store.

This library doesn't support .NET Standard yet.

## sharpcms

This is one of the oldest projects I work on. This is a .NET based content management system with a pretty powerful XSLT template engine. This project was initially built by Mads Hoebi from Denmark (where some good CMS came from.) Me and Thomas Huber took over the project back in 2008 to continue working on it. It is pretty simple to use as a User and simple for a developer to setup a new Web. Currently we are working on a new version based on a data provider system to support more that XML as a data storage. There is also an idea to add support for more template engines.

This project doesn't support .NET Standard yet.