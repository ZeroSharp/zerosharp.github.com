---
layout: post
title: "Replacing a class at runtime using Ninject and Roslyn - Part 1: The Goal"
date: 2014-09-09 10:47
comments: true
categories: [c#, ninject, roslyn, plugins]
description: A powerful and simple plug-in framework for .NET using Ninject and Roslyn. How to replace a class at runtime. Part 1.
---
## The goal ##

{% pullquote %} 
{"How can we replace a given class's code with new code at runtime?"} In particular, how we can we do this while allowing dependency injection and  sidestepping assembly versioning issues.
{% endpullquote %}

Let's say you have bunch of classes like this:

```c# SomeGenerator.cs
public class SomeGenerator : IGenerator
{
    public SomeGenerator(ISomeDependency dependency, IAnotherDependency another)
    {
        ...
    }

    public void Generate()
    {
        ...
        // generate some output
    }
}
```

Now let's assume that you need the ability to modify the behaviour of these classes at runtime without upgrading. And change the dependencies. Without restarting the application.

#### Old school - The MEF approach (and most other plug-in frameworks) ####

One approach would be to place each generator in a separate assembly and then you could load them at runtime. (This was my first effort - _oh how I struggled_). 

You can make use of something like [MEF](http://msdn.microsoft.com/en-us/library/dd460648.aspx) to help with the grunt work, but can still be very complex.

One difficulty is the dependencies. The dependencies are often defined in other assemblies and you have to be very careful to avoid 'dll hell'. It is very easy to get message like:

    Could not load file or assembly 'SomeAssembly, Version=1.2.9.1, Culture=neutral, PublicKeyToken=ad2d246d2bace800' or one of its dependencies. The located assembly's manifest definition does not match the assembly reference.

Or even exceptions like

    Object (of type 'SomeGenerator') is not of type 'SomeGenerator'.

You either have to write your plug-in code so that it is totally independent (i.e., has no dependencies), or you need to resort to a heap of `<bindingRedirect>` tags in your web.config.

Also, with one assembly per format, you can end up with a huge proliferation of assemblies. If you have 50 different formats, that would be 50 assemblies.

#### New school - The Rosyln approach ####

An alternative is to use the compiler-as-a-service features of [Roslyn](http://msdn.microsoft.com/en-gb/vstudio/roslyn.aspx).

Can we upload a modified _SomeGenerator.cs_ and get the it to reference the deployed assemblies and thereby avoid dll hell? With Roslyn we can do this.

If the compilation fails, we can immediately inform the user that the file is not compatible. If it succeeds, we can use it in lieu of the version that was originally deployed.

Also, you do __not__ need separate assemblies for the plug-ins. Your production code contains, within it somewhere a class named `SomeGenerator`. At runtime, we are going to create an in-memory assembly which contains only a single class (still named `SomeGenerator`), but which can nevertheless reference any other class available to the original implementation. Then we will get our IoC container to 'replace' the old generator with the new one.

## The plan ##

* Build an ASP.NET MVC 5 web application. It will use an instance of `HelloWorldGenerator` to generate some output. (This is the _original implementation_).
* Allow a replacement for the `HelloWorldGenerator` class to be uploaded into the application as raw C# code. (This is the _plug-in implementation_.)
* Store the C# code in a database. If the application is restarted, the plug-in code will be reloaded.
* When the output is next requested, compile the new C# class. Any dependencies will be instantiated by the IoC container. If there are any compilation errors, these will be displayed.
* Show that the plug-in class is now being used and the output has changed. The originally shipped `HelloWorldGenerator` class has been replaced by our plug-in.
* Delete the plug-in from the table and show the output has reverted to the default (the originally implementation code).

Over next few posts I'll guide you through building the application and demonstrate the runtime replacement of the generator class.
