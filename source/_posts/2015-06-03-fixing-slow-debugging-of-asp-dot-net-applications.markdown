---
layout: post
title: "Fixing slow debugging of ASP.NET applications"
date: 2015-06-03 21:31
comments: true
categories: [c#, visual studio, devexpress, xaf]
description: Disabling Visual Studio's Browser Link feature improves the debugging experience.
---
For a while I've noticed an annoying slowness when debugging ASP.NET applications from Visual Studio. Just after every page load it takes about a second before the buttons become clickable. I noticed mostly when debugging XAF applications, perhaps because the pages are quite complex.

Turns out the culprit is something called [Browser Link](http://www.asp.net/visual-studio/overview/2013/using-browser-link) which was introduced in Visual Studio 2013. It's enabled by default.

To turn it off you can turn it off from the menu:

{% img /images/blog/browserlink-001.png %}

Or you can add the following to your web.config file.
```xml
<appSettings>
  <add key="vs:EnableBrowserLink" value="false"/>
</appSettings>
```