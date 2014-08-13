---
layout: post
title: "Persisting changes to config files within NuGet packages"
date: 2014-08-13 06:39
comments: true
categories: [c#, nuget, msbuild]
description: Use an MSBuild target to make automatic changes to any xml file.
---
Whenever NuGet updates or restores a NuGet package, the config files within it are overwritten. Here's a method to make sure the changes are reapplied via a [config transform](http://msdn.microsoft.com/en-us/library/dd465326.aspx) whenever the solution is built.

I'm using the NUnit.Runners NuGet packages. To get our coverage tool to play nicely, I need to replace `<supportedRuntime "v2.0.50727">` with `<supportedRuntime "v4.0.30319">` within the _NUnit-console-x86.exe.config_.

Normally, a config transform is for modifying the _web.config_ or _app.config_ files. Here, we need to modify a config file within the _packages_ subdirectory.

In my _.csproj_ file, I have added the following:

```xml
  <PropertyGroup> 
   <NUnitRunnerDir>$(SolutionDir)packages\NUnit.Runners.2.6.3\tools\</NUnitRunnerDir>
  <PropertyGroup>

  <!-- Default NUnit test runner requires a modification to the config file-->
  <UsingTask
    TaskName="TransformXml"
    AssemblyFile="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v12.0\Web\Microsoft.Web.Publishing.Tasks.dll" />

  <Target Name="AfterBuild" Condition="exists('$(NUnitRunnerDir)NUnit-console-x86.exe.config')">
    <TransformXml
        Source="$(NUnitRunnerDir)NUnit-console-x86.exe.config"
        Destination="$(NUnitRunnerDir)NUnit-console-x86.exe.config"
        Transform="$(SolutionDir)UnitTests\Transforms\NUnit-console-x86.exe.CLR4.config" />
  </Target>
```

And the transform file itself looks like this:
{% codeblock lang:xml NUnit-console-x86.exe.CLR4.config %}
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <startup>
    <supportedRuntime version="v4.0.30319" xdt:Transform="Replace" />
  </startup>
</configuration>
{% endcodeblock %}

By the way, [Web.config Transformation Tester](https://webconfigtransformationtester.apphb.com/) is a handy tool!

Now whenever I build the project, the _AfterBuild_ event ensures the supportedRuntime version is set correctly.