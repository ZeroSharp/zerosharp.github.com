---
layout: post
title: "Making XAF reports even better - Part 2"
date: 2013-05-28 11:51
comments: true
categories: [c#, devexpress, xaf, reports]
description: Second part of a series about improving the way DevExpress XafReports are managed.
---
{% pullquote %}
Good news. The conversion is now two-way. Get the source code [from GitHub](https://github.com/ZeroSharp/Xaf_MainDemo_ReportSync). Make sure you have built MainDemo.Reports project.

You will find there are now two T4 transforms in the project. _RepxToCSharp.tt_ is covered in the [Part 1](/making-xaf-reports-even-better-part-1). It searches for any _.repx_ files in the solution and converts the scripts into compilable C#.

The second transform is new. _CSharpToRepx.tt_ copies any changes to the script part back into the original _.repx_ files. Again, there are performance optimisations via checksums to prevent overwriting unchanged files.

{% codeblock lang:text %}
(This is an automatically generated file which should be excluded from version control)

Summary of C# transformation
============================
Total C# files found                                        :  2
  Total reports injected                                    :  1
  Total reports missing                                     :  0
  Total reports skipped because unchanged                   :  1

Time elapsed: 00:00:02.3483264
{% endcodeblock %}

{"Repx files with embedded scripts are now much more maintainable. You can correct syntax errors, refactor, version control, merge versions easily."} You could even write unit tests against the code in the scripts.

{% endpullquote %}

Currently the easiest way of running these scripts is to open them and save them with `Ctrl+S`. This is because T4 templates were originally designed as a Visual Studio tool. 

In the future I'm hoping to improve the integration further. There are ways of including the transformations into the build instead, most of which are covered in [a blog post by Mr T4, Oleg Sych](http://www.olegsych.com/2010/04/understanding-t4-msbuild-integration). I like the idea of it being a NuGet package that can be easily added to any XAF project, but there's a but I'll need some more time to work out how best to achieve this. 

## Basic usage summary ##

Until then, here are some basic usage instructions.

* Add the T4 Toolbox extension to Visual Studio
* Add a copy of the MainDemo.Reports project to your own solution
* Make sure you build it before running the transforms
* Open _RepxToCSharp.tt_ in Visual Studio.
* Save it with `Ctrl+S` to run the transform. It will search all the folders in your Solution for _.repx_ files and add corresponding C# classes.
* Make any changes you like to the script section (anything outside of `// -- Start of embedded scripts --` and ``// -- End of embedded scripts --``) will be ignored.
* Open _CSharpToRepx.tt_ and run it with `Ctrl+S`. The changes will be saved back to the corresponding _.repx_.

## Even more power? ##

You may notice that if you reload the MainDemo.Reports project, you can now see `View in Designer` in the context menu when you right-click on the _.repx.cs_ file.

{% img /images/blog/xaf-report-sync-003.png %}

Let's click it and see what happens. It opens directly in Visual Studio (like an `XtraReport`).

{% img /images/blog/xaf-report-sync-004.png %}

Now, this is all highly experimental. You can see there are some warnings... Also, there is no connection with XPO, so the _Preview_ is always empty. 

That said, it doesn't seem like too much of a stretch to eventually allow far more Visual Studio integration for XAF reports...

