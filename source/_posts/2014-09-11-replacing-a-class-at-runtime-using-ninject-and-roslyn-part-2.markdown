---
layout: post
title: "Replacing a class at runtime using Ninject and Roslyn - Part 2: The Solution"
date: 2014-09-11 09:18
comments: true
categories: [c#, ninject, roslyn, plugins]
description: A powerful and simple plug-in framework for .NET using Ninject and Roslyn. Part 2.
---
Previously: [Part 1: The Goal](/replacing-a-class-at-runtime-using-ninject-and-roslyn-part-1/)

## The solution ##

The code for the example is [available on GitHub](https://github.com/ZeroSharp/RoslynPlugins).

#### How it looks ####

So here's the Hello World page in production:

{% img /images/blog/roslyn-plugins-001.png %}.

We navigate to the plugins view and create a new replacement for the HelloWorldGenerator:

{% img /images/blog/roslyn-plugins-002.png %}.

Without restarting, we can return to the HelloWorld page and see that the new class is being used because the output has changed.

{% img /images/blog/roslyn-plugins-003.png %}.

If you delete the row from the plugins page, the behaviour reverts to the original implementation (the code that was originally shipped with production).

#### Basic project setup ####

First, I created a new ASP.NET MVC 5 application. I added a HelloWorldContrroller and a View. I added a `Plugin` model and corresponding views. To get started I followed the tutorial here (http://www.asp.net/mvc/tutorials/mvc-5/introduction/getting-started). Once I had the basics in place, I added the following NuGet packages.

**Stable**

* EntityFramework
* Ninject
* Ninject.MVC5
* Ninject.Conventions

**Pre-release**

* Microsoft.CodeAnalysis.CSharp

The _Microsoft.CodeAnalysis.CSharp_ is the 'Roslyn' package. It is still in beta, so you have to switch to the pre-release.

Next we'll look at [the dependency injection part](/replacing-a-class-at-runtime-using-ninject-and-roslyn-part-3/) in more detail.