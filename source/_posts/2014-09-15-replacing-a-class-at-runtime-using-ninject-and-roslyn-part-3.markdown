---
layout: post
title: "Replacing a class at runtime using Ninject and Roslyn - Part 3: Dependency Injection"
date: 2014-09-15 08:01
comments: true
categories: [c#, ninject, roslyn, plugins]
description: A powerful and simple plug-in framework for .NET using Ninject and Roslyn. Part 3.
---
Previously

* [Part 1: The Goal](/replacing-a-class-at-runtime-using-ninject-and-roslyn-part-1/)
* [Part 2: The Solution](/replacing-a-class-at-runtime-using-ninject-and-roslyn-part-2/)

## Dependency injection ##

The first trick is to use dependency injection to create any instance of the `HelloWorldGenerator` class. Then if we need to add a new dependency to the class, we can just add it to the constructor without breaking anything.

```c# HelloWorldGenerator.cs
public class HelloWorldGenerator : IGenerator
{
    public HelloWorldGenerator(
        ISomeDependency dependency, 
        IAnotherDependency another, 
        INewDependency new // a new dependency!!!)
    {
        ...
    }
```

We'll use Ninject here, but you ought to be able to achieve the same with any dependency injection framework.

So normally, we'd have a binding something like:

```c# 
Bind<IGenerator>().To<HelloWorldGenerator>();
```

Instead we'll replace this with a binding to a factory method instead.

```c# 
Bind<IGenerator>().ToMethod(context => CreatePluginInstance(context));
```

The `CreatePluginInstance(context)` method will try to find an `IGenerator` class within any available plug-ins. If it finds one, it will ask the Ninject framework to create an instance of the plug-in class. Otherwise it falls back to the default type (the original implementation of the generator). The `PluginLocator` it is responsible for searching any runtime-compiled assemblies for candidate plug-ins. We'll look at it in more detail later.

```c# 
private IGenerator CreatePluginInstance(IContext context)
{
    var pluginLocator = context.Kernel.Get<PluginLocator>();
    Type roslynPluginType = pluginLocator.Locate<IGenerator>();
    
    /// if we found a plug-in, create an instance of it
    if (roslynPluginType != null)
        return (IGenerator)context.Kernel.Get(roslynPluginType);
    else ///otherwise create an instance of the original implementation
        return context.Kernel.Get<HelloWorldGenerator>();
}
```

#### By convention ####
Of course, you might have dozens of `IGenerator` descendants, in which case you can use [Ninject's convention-based binding module](https://github.com/ninject/ninject.extensions.conventions). (Don't forget to add it with NuGet). My version looks something like the following.

```c#
/// If you have a lot of IGenerator subclasses, you can use Ninject's
/// convention based module.
/// 
///   For each Generator, bind to IGenerator. 
///   For example, Bind<IGenerator>.To<SomeGenerator>();
/// 
Kernel.Bind(scanner => scanner
    .FromThisAssembly()
    .SelectAllClasses()
    .InheritedFrom(typeof(IGenerator))
    .BindToPluginOtherwiseDefaultInterfaces()); //This is a custom extension method (see below)
```

```c#
    public static class ConventionSyntaxExtensions
    {
        public static IConfigureSyntax BindToPluginOtherwiseDefaultInterfaces(this IJoinFilterWhereExcludeIncludeBindSyntax syntax)
        {
            return syntax.BindWith(new DefaultInterfacesBindingGenerator(new BindableTypeSelector(), new PluginOtherwiseDefaultBindingCreator()));
        }
    }

    /// <summary>
    /// Returns a Ninject binding to a method which returns the plug-in type if one exists, otherwise returns the default type.
    /// </summary>
    public class PluginOtherwiseDefaultBindingCreator : IBindingCreator
    {
        public IEnumerable<IBindingWhenInNamedWithOrOnSyntax<object>> CreateBindings(IBindingRoot bindingRoot, IEnumerable<Type> serviceTypes, Type implementationType)
        {
            if (bindingRoot == null)
            {
                throw new ArgumentNullException("bindingRoot");
            }

            return !serviceTypes.Any()
             ? Enumerable.Empty<IBindingWhenInNamedWithOrOnSyntax<object>>()
             : new[] { bindingRoot.Bind(serviceTypes.ToArray()).ToMethod(context => context.Kernel.Get(context.Kernel.Get<PluginLocator>().Locate(serviceTypes) ?? implementationType)) };
        }
    }
```

Next we'll look at the Roslyn part in more detail.