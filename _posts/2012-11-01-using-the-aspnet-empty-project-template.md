---
layout: post
title: "Using the ASP.Net Empty Project Template"
description: ""
category: ASP.Net
tags: [asp.net, visual studio]
---
{% include JB/setup %}

##Getting yourself out of a jam once you've built on top of the ASP.Net Empty Project Template

With the launch of ASP.Net v4.5 and MVC4/WebAPI etc, the ASP.Net team took the admirable step to making a "single ASP.Net". The theory being that
the different project types available to users in Visual Studio rail-roaded the user to making big choices about the application they were building
before they'd even really started. Whilst the different template types were great in terms of saving the user some initial configuration work to do
when creating a project (e.g. referncing the relevant DLLs, setting up handlers in the web.config file), it became difficult to mix project types.

By creating a single ASP.Net, Visual Studio could offer users the ability to have a "plain" ASP.Net website project, but they could then add MVC4 capabilities
by downloading the [MVC4 Nuget package](http://nuget.org/packages/Microsoft.AspNet.Mvc). When tasked with a new website build, I thought I could put this to the test,
as initially the website only needed to handle requests for PDF files, which I knew I could serve using a HttpHandler. I knew there was no need for the extra bloat
and code involved in using an MVC controller, so decided to opt for the ASP.Net Empty Project template, knowing that I should be able to get MVC added later into the same 
project if/when needed. So far so good, the website was launched, PDFs were served to users, everybody was happy.

