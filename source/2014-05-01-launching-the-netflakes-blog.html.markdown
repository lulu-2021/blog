---
title: launching the netflakes blog
date: 2014-05-01 20:00 EST
tags: azure, azure-management-client-library
---

## Launching my new code blog on github

#### This is my attempt to document my journey whilst developing with using C# and some Ruby and others

Lately I have been working with the Azure platform a fair bit and with the recent release of the [Windows Azure Management Libraries](http://www.bradygaster.com/post/getting-started-with-the-windows-azure-management-libraries).

Prior to this I have used powershell to interact with the Windows Azure Rest API, but this new set of Client Libraries is so much more fun.

My initial solution is available on my [github.com/netflakes](https://github.com/netflakes/AzureManagement) account. Sofar I have a project that gathers stats on the cloud services and virtual machines and can also download sets of RDP files for all or for Cloud services.

Following my passion for TDD I have a set of unittests for each method and this will grow. Logging and configuration is isolated from the main code via DI. There is a command line app that will be able to call each method in the class library. Following on from this I intend to add different output/storage mechanisms also via DI that can be plugged in when needed.

At my work we have a large number of subscriptions and associated services for the many projects we are involved in and this is becoming ever more complex to manage. Automating this is the only way to keep a handle on things.

We also have the need to dissseminate reports and information to developers and managers and I can see some benefit for adding a small webservice. Once we can cache all the information retrieved from Azure locally we can make this available.

[Nancy](http://nancyfx.org) seems like a great fit for the webservice that will be added.