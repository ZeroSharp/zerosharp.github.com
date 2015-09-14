---
layout: post
title: "Migrating a large web application from XAF 12.1 to 15.1 - Part 1"
date: 2015-09-14 10:01
comments: true
categories: [c#, devexpress, xaf]
description: Describing some of the challenges encountered in upgrading to the latest DevExpress expressAppFramework. In this part I describe the general approach and have some early success.
---
I am the principal software architect for a treasury application in use by over 100 large multinational corporates. Upgrades are generally met with reluctance in the enterprise world and so we've been somewhat stuck on an old version of the expressAppFramework. 

Recently I've been pushing to move to the newer version and I have spent about three weeks migrating the substantial code base to XAF 15.1 and .NET 4.6.

The steps are:

- Run the project converter tool. 
- Try to compile. Identify the errors which are easily fixable (check with the 'breaking changes' documents from DevExpress.)
- When in doubt, compare the libraries with a decompiler like .NET Reflector.
- Refactor where necessary (ensure you have unit tests for the behaviour you are changing).

On this last point, my whole approach to refactoring has been shaped by Michael C. Feather's book [_Working Effectively with Legacy Code_](http://amzn.com/0131177052). Highly recommended for anyone maintaining complex applications regardless of whether the code is legacy or not...

I was pleasantly surprised that I was very quickly able to get everything to compile and even run. The layout was not correct, but a lot of things worked straight away.

I then had to spend some time restoring the customisations we'd made to the default ASP.NET layout. In XAF 12.1 these were applied directly to _default.aspx.cs_ and _dialog.aspx.cs_, but in 15.1 these no longer exist. Instead, you can customise layout by providing your own alternate templates. I was expecting this to be much harder, but by [following the instructions in the documentation](https://documentation.devexpress.com/#eXpressAppFramework/CustomDocument112696) I managed to restore our layout quite easily.

I still had a lot of failing unit tests. One seemingly minor change to XAF validation proved to be a lot of work to fix in our code. 
Since 13.1, the XAF `Validator` class now requires an `IObjectSpace` parameter in the constructor. This was by far the biggest problem to fix and is the subject of the next blog post.
