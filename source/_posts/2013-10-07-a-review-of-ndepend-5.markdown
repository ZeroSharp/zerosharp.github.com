---
layout: post
title: "A review of NDepend 5"
date: 2013-10-02 17:54
comments: true
categories: [c#, ndepend, devexpress, xaf]
description: A review of NDepend version 5.
---

NDepend is a commercial static analysis tool for .NET managed code. It's been around a long time (since 2004!). Version 5 was just released and in this post I'm going to try it out on the DevExpress MainDemo.

{% pullquote %}
In the past I have always thought of NDepend as a complex tool. I was never sure where to start. {"In version 5, a lot of work has been done to improve the learning curve."} The installation process is easy and a wizard very quickly points you in the right direction.

After [downloading the v5 trial](http://ndepend.com/NDependDownload.aspx) and running the installation you get to the following screen.
{% endpullquote %}

{% img /images/blog/ndepend/ndepend-001.png %}

You cannot miss that big red arrow. In fact, it's even animated in the actual product. Click on it and choose a Visual Studio solution to analyse. I'm going to navigate to the DevExpress MainDemo 13.1.

So long as the project has been built at some point, NDepend works out what to analyse (otherwise you'll get helpful warnings). 

{% img /images/blog/ndepend/ndepend-003.png %}

It is very fast and we get to this help screen.

{% img /images/blog/ndepend/ndepend-004.png %}

Choose the dashboard.

{% img /images/blog/ndepend/ndepend-005.png %}

Now that's a lot of information, but it's all well presented and easy to navigate. Let's focus on the worst: I can see _2 Critical Rules Violated_. Drill down to find out more:

{% img /images/blog/ndepend/ndepend-009.png %}

It's complaining that we have classes with duplicate names in our projects. In the pane on the left we can see what they are: 3 types named `MainDemoWinApplication`, 3 named `Program`, _etc._ And sure enough there are: _MainDemo.Win_, _MainDemo.Win.Mdi_ and _MainDemo.Win.Ribbon_ all duplicate those class names.

We can also see 2 `TaskAnalysis1LayoutUpdater` types and a quick search reveals that there's one in the web module and another in the win module.

So NDepend has correctly discovered some potential issues. As we XAF fans know, this one is not really a problem, because those modules are never loaded into the same AppDomain, but nevertheless the information is accurate and relevant.

Lets have a brief look at the other screens. The dependency graph:

{% img /images/blog/ndepend/ndepend-006.png %}

With a million options.

{% img /images/blog/ndepend/ndepend-010.png %}

A dependency matrix.

{% img /images/blog/ndepend/ndepend-007.png %}

A metrics view showing class and assembly sizes.

{% img /images/blog/ndepend/ndepend-008.png %}

All with reams of helpful documentation.

{% img /images/blog/ndepend/ndepend-011.png %}

Overall, the tool felt fast, responsive and stable. I've focused on the user interface aspects, but there is so much more. Some things you can do: 

* Create your own analysis rules
* [Query your own code with LINQ](http://www.ndepend.com/Doc_CQLinq_Syntax.aspx#Edition) 
* Run analysis from [the command line](http://www.ndepend.com/NDependConsole.aspx)
* Run from within Visual Studio
* Add output to continuous integration ([TeamCity](http://www.ndepend.com/Doc_CI_TeamCity.aspx), [CruiseControl.NET](http://www.ndepend.com/Doc_CI_CCNet.aspx), etc.)
* Track [trends and progress over time](http://www.ndepend.com/Doc_Trend.aspx)

NDepend shines at providing a high-level overview of code quality and as such it is a very useful addition to any developer's toolkit. There are some scenarios where NDepend would be particularly useful:

* For a developer joining a mature project.
* For a senior developer looking to track progress on a refactoring drive.
* For helping evaluate the quality of an open source third party library.

In this quick review, I'm not going deep enough to say anything about whether the DevExpress MainDemo is good code or not - it's just a sample project I happen to be quite familiar with. It might be interesting to unleash NDepend on the full DevExpress source code and maybe one day I'll get around to writing a future post about that.

With regard to my own projects, I feel I'm so familiar with them that I ought to be aware of most of the recommendations NDepend is likely to make, but I'll give it a spin and see what comes out...
