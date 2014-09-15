---
layout: post
title: "Replacing a class at runtime using Ninject and Roslyn - Part 4: Roslyn"
date: 2014-09-17 08:08
comments: true
categories: [c#, ninject, roslyn, plugins]
description: A powerful and simple plug-in framework for .NET using Ninject and Roslyn. Part 4.
published: false
---
Previously

* [Part 1: The Goal](/replacing-a-class-at-runtime-using-ninject-and-roslyn-part-1/)
* [Part 2: The Solution](/replacing-a-class-at-runtime-using-ninject-and-roslyn-part-2/)
* [Part 3: Dependency Injection](/replacing-a-class-at-runtime-using-ninject-and-roslyn-part-3/)

## Roslyn ##

I haven't covered every detail of the Roslyn side of things - download the source code to see it all working. Here I'll just cover the main classes.

The `PluginSnippetCompiler` takes a string (for instance, the contents of an uploaded raw C# file) and converts it into an in-memory assembly with the same assembly references as the main project.

The Roslyn compiler is still in beta, and the Microsoft team have recently removed some syntactic sugar which made this task easier to read but the code below works with version 0.7.0.0.

```c#
public class PluginSnippetCompiler
{
    public PluginSnippetCompiler(IAssemblyReferenceCollector assemblyReferenceCollector)
    {
        if (assemblyReferenceCollector == null)
            throw new ArgumentNullException("assemblyReferenceCollector");

        _AssemblyReferenceCollector = assemblyReferenceCollector;
    }

    private readonly IAssemblyReferenceCollector _AssemblyReferenceCollector;

    private IEnumerable<Diagnostic> _Diagnostics = Enumerable.Empty<Diagnostic>();

    public IEnumerable<Diagnostic> Errors
    {
        get
        {
            return _Diagnostics
                .Where(d => d.Severity == DiagnosticSeverity.Error);
        }
    }

    public IEnumerable<Diagnostic> Warnings
    {
        get
        {
            return _Diagnostics
                .Where(d => d.Severity == DiagnosticSeverity.Warning);
        }
    }

    private string GetOutputAssemblyName(string name)
    {
        return String.Format("RoslynPlugins.Snippets.{0}", name);
    }

    /// <summary>
    /// Compiles source code at runtime into an assembly. The assembly will automatically include all
    /// the same assembly references as the main RoslynPlugins assembly, so you can call any function which is
    /// available from within the deployed RoslynPlugins. Compilation errors and warnings can be obtained from 
    /// the Errors and Warnings properties.
    /// </summary>
    /// <param name="name">The name of the class, e.g., HelloWorldGenerator</param>
    /// <param name="script">Source code such as the contents of HelloWorldGenerator.cs</param>
    /// <returns>The compiled assembly in memory. If there were errors, it will return null.</returns>
    public Assembly Compile(string name, string script)
    {
        if (name == null)
            throw new ArgumentNullException("name");

        if (script == null)
            throw new ArgumentNullException("script");

        string outputAssemblyName = GetOutputAssemblyName(name);

        var defaultImplementationAssembly = typeof(HelloWorldGenerator).Assembly;
        var assemblyReferences = _AssemblyReferenceCollector.CollectMetadataReferences(defaultImplementationAssembly);

        // Parse the script to a SyntaxTree
        var syntaxTree = CSharpSyntaxTree.ParseText(script);

        // Compile the SyntaxTree to an in memory assembly
        var compilation = CSharpCompilation.Create(outputAssemblyName,
            new[] { syntaxTree },
            assemblyReferences,
            new CSharpCompilationOptions(OutputKind.DynamicallyLinkedLibrary));

        using (var outputStream = new MemoryStream())
        {
            using (var pdbStream = new MemoryStream())
            {
                // Emit assembly to streams. Throw an exception if there are any compilation errors
                var result = compilation.Emit(outputStream, pdbStream: pdbStream);

                // Populate the _diagnostics property in order to read Errors and Warnings
                _Diagnostics = result.Diagnostics;

                if (result.Success)
                {
                    return Assembly.Load(outputStream.ToArray(), pdbStream.ToArray());
                }
                else
                {
                    return null;
                }
            }
        }
    }
}
```

In this demo, I have not included any user feedback about compilation errors, but they are easily obtainable from the `Errors` and `Warnings` properties. If there is an error, the plug-in will be ignored and the default implementation will be used.

The class above depends on an `AssemblyReferenceCollector` which is responsible for enumerating the references to add to the runtime generated plug-in assembly.

```c#
public class AssemblyReferenceCollector : IAssemblyReferenceCollector
{
    public IEnumerable<MetadataReference> CollectMetadataReferences(Assembly assembly)
    {
        var referencedAssemblyNames = assembly.GetReferencedAssemblies();

        var references = new List<MetadataReference>();
        foreach (AssemblyName assemblyName in referencedAssemblyNames)
        {
            var loadedAssembly = Assembly.Load(assemblyName);
            references
                .Add(new MetadataFileReference(loadedAssembly.Location));
        }

        references
            .Add(new MetadataFileReference(assembly.Location)); // add a reference to 'self', i.e., NetMWC

        return references;
    }
}
```

#### Connecting the pieces ####

We need the `PluginLocator` class to connect the Ninject resolution root to the runtime-generated assembly (if one exists). It just looks for classes with the correct interface `IGenerator` within the `PluginAssemblyCache`.

Here's out it looks:

```c#
public class PluginLocator
{
    public PluginLocator(PluginAssemblyCache pluginAssemblyCache)
    {
        if (pluginAssemblyCache == null)
            throw new ArgumentNullException("pluginAssemblyCache");

        _PluginAssemblyCache = pluginAssemblyCache;
    }

    private readonly PluginAssemblyCache _PluginAssemblyCache;

    public Type Locate<T>()
    {
        return Locate(new[] { typeof(T) });
    }

    protected Type Locate(IEnumerable<Type> serviceTypes)
    {
        var implementingClasses = AssemblyExplorer.GetImplementingClasses(_PluginAssemblyCache.GetAssemblies(), serviceTypes);

        if (implementingClasses.Any())
        {
            if (implementingClasses.Count() > 1)
                throw new Exception("More than one plugin class found which implements " + String.Join(" + ", serviceTypes.Select(t => t.ToString())));
            else
                return implementingClasses.Single();
        }
        return null;
    }
}
```

The `PluginAssemblyCache` has the following dependencies:

* an `IPluginSnippetProvider` which (in this case) reads the existing snippets from the database
* a `PluginLoader` which uses the above `PluginSnippetCompiler` to convert a snippet into a runtime assembly and display any compilation errors on failure.

```c#
    /// <summary>
    /// This class maintains a list of runtime-compiled in memory assemblies loaded from the plugins
    /// available via the provider. It is a singleton class.
    /// </summary>
    public class PluginAssemblyCache
    {
        public PluginAssemblyCache(IPluginSnippetProvider pluginSnippetProvider, PluginLoader pluginLoader)
        {
            if (pluginSnippetProvider == null)
                throw new ArgumentNullException("pluginSnippetProvider");
            _PluginSnippetProvider = pluginSnippetProvider;

            if (pluginLoader == null)
                throw new ArgumentNullException("pluginLoader");
            _PluginLoader = pluginLoader;
        }

        private class CacheEntry
        {
            public string Name { get; set; }
            public Version Version { get; set; }
            public Assembly Assembly { get; set; }
        }

        private readonly IPluginSnippetProvider _PluginSnippetProvider;
        private readonly PluginLoader _PluginLoader;

        private List<CacheEntry> _Cache = new List<CacheEntry>();

        private void Add(string name, string version, Assembly assembly)
        {
            var cacheEntry =
                new CacheEntry()
                {
                    Name = name,
                    Version = new Version(version),
                    Assembly = assembly
                };
            _Cache.Add(cacheEntry);
        }

        private void RefreshCache()
        {
            var pluginScriptContainers = _PluginSnippetProvider.GetPlugins();

            // Add a new assembly for any new or updated plugin
            foreach (var pluginScriptContainer in pluginScriptContainers)
            {
                var name = pluginScriptContainer.Name;
                var version = pluginScriptContainer.Version;
                if (!_Cache.Any(a => a.Name == name && a.Version == new Version(version)))
                {
                    var assembly = _PluginLoader.Load(pluginScriptContainer);
                    Add(name, version, assembly);
                }
            }

            // Remove any assemblies which we no longer have a plugin for.
            _Cache
                .RemoveAll(cacheEntry =>
                    !pluginScriptContainers
                        .Select(plugin => plugin.Name)
                        .Contains(cacheEntry.Name));
        }

        public IEnumerable<Assembly> GetAssemblies()
        {
            RefreshCache();

            // Return only the assemblies with the highest version numbers
            return _Cache
                .GroupBy(d => d.Name)
                .Select(g => g
                        .OrderByDescending(d => d.Version)
                        .First()
                        .Assembly);
        }
    }
```

So whenever the `SomeGenerator` class is resolved by Ninject, it will now 

* Check whether there are any new plug-ins and compile them into runtime assemblies and add them to the `PluginAssemblyCache`. 
* Then the `PluginLocator` will search these assemblies for a newer version of `SomeGenerator`. 
* If it finds one, it will be resolved along with any constructor dependencies, otherwise it will use the original `SomeGenerator`.

#### Version numbers ####

The version number of the plug-in is a key part of our solution. Let's say you have version 1.0 in production. Then you fix some bugs in staging (version 1.1). You create a plug-in from this staging code and upload it into production. Then much later, you decide to upgrade production to 1.2. Then, with the query in `GetAssemblies()`, the 1.1 plug-in will automatically be ignored and be superseded by whatever was shipped with 1.2 __since that is newer code__. So we do not have to remember to remove obsolete plug-ins after an upgrade - they will automatically be ignored because of the version number.

#### Security ####

Obviously, security is a chief concern and you may have to secure the plug-ins. In this demo project, I just created a simple view for the `IPlugin` object, but in our production environment we handle the creation of plug-ins differently. We use a combination of role-based security (to control who has permission to upload plugins) and encryption with checksumming. No user can directly enter arbitrary code - instead, we send the user a zip file which contains the code (encrypted), the version number and a checksum and our application verifies the checksum and builds the `IPlugin` object from the contents of the zip. A Powershell script running on our build server is responsible for creating the checksummed plug-in directly from the source code used in our staging environment.

## Conclusions - the ultimate plug-in framework?##

The strength of this approach is that it is easy to maintain while being extremely versatile. In our case, it provides us with the ability to restrict the number of major releases approximately one per annum while catering for the inevitable little fixes to output formats and reports.

In the example we replaced an existing class, but it would be straightforward to add the concept of _discovery_ and use the same Roslyn features to make any _new_ plug-in classes available to your application. Ninject, makes it easy to instantiate, say, every implementor of `IGenerator`, so you could enumerate all available plug-ins instead of replacing a single one.

So here's a plugin framework which is very flexible and very powerful without many of the versioning headaches of MEF or MAF. It's also easy to maintain, since the plug-in code is identical to the 'normal' code in staging (just packaged, delivered and compiled in a different way to production).