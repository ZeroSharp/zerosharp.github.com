---
layout: post
title: "My essential Visual Studio tools and extensions"
date: 2018-09-10 15:45
comments: true
categories: [visualstudio]
---

This is part two of a series of posts about the software and tools I find invaluable. See [part 1](/essential-applications) and [part 3](/my-essential-web-applications).

## CodeRush
I've been using DevExpress [CodeRush](https://www.devexpress.com/products/coderush/) since 2005. Check out [this video tutorial](https://youtu.be/v5-MVSoqCnU) for a lightening tour of a lot of the features, and look at the DevExpress youtube channel for [a load of other tutorials](https://www.youtube.com/playlist?list=PL8h4jt35t1wgawacCN9wmxq1EN36CNUGk).

## NCrunch
[NCrunch](https://www.ncrunch.net/) provides continuous testing for Visual Studio. When I make any change to my code which breaks a unit test, the NCrunch risk status goes red a few seconds later, even without recompiling. I get immediate feedback for any breaking change, so long as I have a test for it. Not only does it encourages me and my team to write good tests, but it allows us to make new changes and 
refactor with confidence.

{% img /images/blog/ncrunch-progress-bar.png %}

## Redgate .NET Reflector Developer Bundle
When the going gets tough, the [ANTS Performance Profiler](https://www.red-gate.com/products/dotnet-development/ants-performance-profiler/index) and [ANTS Memory Profiler](https://www.red-gate.com/products/dotnet-development/ants-memory-profiler/index) have helped me find some of the hardest bugs I've come across. 

Also, while there are several good .NET decompilers, many of them free, I got used to [.NET Reflector](https://www.red-gate.com/products/dotnet-development/reflector/index) which comes part of this bundle.

## T4
I use code generation to automatically generate templates for unit tests for validation rules and [T4](https://docs.microsoft.com/en-us/visualstudio/modeling/code-generation-and-t4-text-templates?view=vs-2017) and the [T4 Toolbox](https://marketplace.visualstudio.com/items?itemName=OlegVSych.T4ToolboxforVisualStudio2017) fit the bill nicely.

For a more advanced use case, see my post about automatically [converting DevExpress v1 report scripts](/making-xaf-reports-even-better-part-1/) to check for compilation errors and allow for unit testing of report scripts. 

## AWS Toolkit
I always install the [Amazon Web Services Toolkit](https://aws.amazon.com/visualstudio/) to make it easy to spin up test servers in the Amazon cloud.

## Next up
My [essential web applications and iPhone apps](/my-essential-web-applications).

