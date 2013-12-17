---
layout: post
title: "Glimpse with DevExpress XAF"
date: 2013-12-17 12:41
comments: true
categories: [c#, devexpress, xaf]
description: How to add MiniGlimpse to a DevExpress XAF web application.
---
I have finally got around to getting Glimpse working with XAF. Glimpse is an amazing extensible ASP.NET plug-in which gives you valuable information about what is going on within your server in production. It's also very pretty.

Let's jump right in and have a look at what XAF looks like with Glimpse running.

{% img /images/blog/xaf-with-glimpse-016.png %}

That banner along the bottom of the screen is the Glimpse heads up display (HUD). Hovering over various sections of it pops up more information:

{% img /images/blog/xaf-with-glimpse-019.png %}

{% img /images/blog/xaf-with-glimpse-018.png %}

{% img /images/blog/xaf-with-glimpse-017.png %}

If you click on the Glimpse icon in the bottom right, you get even more goodies. Here's the _Timeline_ view.

{% img /images/blog/xaf-with-glimpse-011.png %}

And there are many other tabs available. The _Configuration_ tab shows the contents of the _web.config_ file.
{% img /images/blog/xaf-with-glimpse-009.png %}
Here's the _Control Tree_ tab:
{% img /images/blog/xaf-with-glimpse-010.png %}
The _Page Life Cycle tab:
{% img /images/blog/xaf-with-glimpse-005.png %}
The _Request_ tab:
{% img /images/blog/xaf-with-glimpse-004.png %}
The _Session_ tab:
{% img /images/blog/xaf-with-glimpse-003.png %}
And the _Trace_ tab including the DevExpress log items that were added to the trace during this page load:
{% img /images/blog/xaf-with-glimpse-002.png %}
As you can see, this is a very valuable glimpse into the server which can be turned on as needed in production.

## Adding Glimpse to an XAF application ##

First install the Glimpse Nuget package into the project.

{% img /images/blog/xaf-with-glimpse-001.png %}

The Nuget installation will make various incorrect changes to the web.config. The corrected sections are below:

First, add Glimpse to the `<configSections>`.
```diff web.config
  <configSections>
    <sectionGroup name="devExpress">
      <section name="compression" requirePermission="false" type="DevExpress.Web.ASPxClasses.CompressionConfigurationSection, DevExpress.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
      <section name="themes" type="DevExpress.Web.ASPxClasses.ThemesConfigurationSection, DevExpress.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
      <section name="settings" type="DevExpress.Web.ASPxClasses.SettingsConfigurationSection, DevExpress.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
    </sectionGroup>
+   <section name="glimpse" type="Glimpse.Core.Configuration.Section, Glimpse.Core" />
  </configSections>
```

Next, `<system.webserver>` should look like this:
```diff
  <system.webServer>
    <handlers>
      <add name="TestControls.axd_*" path="TestControls.axd" verb="*" type="DevExpress.ExpressApp.Web.TestScripts.TestScriptsManager, DevExpress.ExpressApp.Web.v13.2, Version=13.2.5.0, culture=neutral, PublicKeyToken=b88d1754d700e49a" preCondition="integratedMode" />
      <add name="ImageResource.axd_*" path="ImageResource.axd" verb="*" type="DevExpress.ExpressApp.Web.ImageResourceHttpHandler, DevExpress.ExpressApp.Web.v13.2, Version=13.2.5.0, culture=neutral, PublicKeyToken=b88d1754d700e49a" preCondition="integratedMode" />
      <add name="SessionKeepAliveReconnectHttpHandler" verb="*" path="SessionKeepAliveReconnect.aspx*" type="DevExpress.ExpressApp.Web.SessionKeepAliveReconnectHttpHandler, DevExpress.ExpressApp.Web.v13.2, Version=13.2.5.0, culture=neutral, PublicKeyToken=b88d1754d700e49a" preCondition="integratedMode" />
      <add name="WebWindowTemplateHttpHandler" verb="*" path="*.aspx" type="DevExpress.ExpressApp.Web.WebWindowTemplateHttpHandler, DevExpress.ExpressApp.Web.v13.2, Version=13.2.5.0, culture=neutral, PublicKeyToken=b88d1754d700e49a" preCondition="integratedMode" />
      <add name="ASPxUploadProgressHandler" verb="GET,POST" path="ASPxUploadProgressHandlerPage.ashx" type="DevExpress.Web.ASPxUploadControl.ASPxUploadProgressHttpHandler, DevExpress.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" preCondition="integratedMode" />
      <add name="ReportExportResource.axd_*" preCondition="integratedMode" verb="*" path="ReportExportResource.axd" type="DevExpress.ExpressApp.Reports.Web.ReportExportHttpHandler, DevExpress.ExpressApp.Reports.Web.v13.2, Version=13.2.5.0, culture=neutral, PublicKeyToken=b88d1754d700e49a" />
 +    <add name="Glimpse" path="glimpse.axd" verb="GET" type="Glimpse.AspNet.HttpHandler, Glimpse.AspNet" preCondition="integratedMode" />
    </handlers>
    <validation validateIntegratedModeConfiguration="false" />
    <modules>
      <add name="ASPxHttpHandlerModule" type="DevExpress.Web.ASPxClasses.ASPxHttpHandlerModule, DevExpress.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
 +    <add name="Glimpse" type="Glimpse.AspNet.HttpModule, Glimpse.AspNet" preCondition="integratedMode" />
    </modules>
  </system.webServer>
```

NuGet added some incorrect definitions to `system.web`. Make sure to restore this section to the DevExpress default:

```xml
  <system.web>
    <httpRuntime requestValidationMode="2.0" />
    <sessionState mode="InProc" timeout="2" />
    <httpHandlers configSource="HttpHandlers.Web.Config" />
    <httpModules configSource="HttpModules.Web.Config" />
```

And now we'll add the corrected changes to `HttpHandlers.Web.Config` and `HttpModule.Web.Config`

```diff HttpHandlers.Web.Config
<?xml version="1.0" encoding="utf-8"?>
<httpHandlers>
+ <add verb="GET" path="glimpse.axd" type="Glimpse.AspNet.HttpHandler, Glimpse.AspNet" />
  <add verb="*" path="TestControls.axd" type="DevExpress.ExpressApp.Web.TestScripts.TestScriptsManager, DevExpress.ExpressApp.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
  <add verb="*" path="ImageResource.axd" type="DevExpress.ExpressApp.Web.ImageResourceHttpHandler, DevExpress.ExpressApp.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
  <add verb="*" path="SessionKeepAliveReconnect.aspx*" type="DevExpress.ExpressApp.Web.SessionKeepAliveReconnectHttpHandler, DevExpress.ExpressApp.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
  <add verb="*" path="*.aspx" type="DevExpress.ExpressApp.Web.WebWindowTemplateHttpHandler, DevExpress.ExpressApp.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
  <add verb="GET,POST" path="ASPxUploadProgressHandlerPage.ashx" validate="false" type="DevExpress.Web.ASPxUploadControl.ASPxUploadProgressHttpHandler, DevExpress.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
  <add verb="*" path="ReportExportResource.axd" type="DevExpress.ExpressApp.Reports.Web.ReportExportHttpHandler, DevExpress.ExpressApp.Reports.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
</httpHandlers>
```

```xml HttpModules.Web.Config
<?xml version="1.0" encoding="utf-8"?>
<httpModules>
+ <add name="Glimpse" type="Glimpse.AspNet.HttpModule, Glimpse.AspNet" />
  <add name="ASPxHttpHandlerModule" type="DevExpress.Web.ASPxClasses.ASPxHttpHandlerModule, DevExpress.Web.v13.2, Version=13.2.5.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
</httpModules>
```

Next, there is [an outstanding problem](https://github.com/Glimpse/Glimpse/issues/632) with the current release version of the Glimpse ASP.NET NuGet package (1.6.0) which prevents it from working with the development webserver. (Apparently it's fixed in 1.7.0 which should be released soon). If you try to run MainDemo.Web you will get the following error:

    Type 'System.Web.HttpContextWrapper' in assembly 'System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' is not marked as serializable.

However it works fine against IIS Express (or full IIS). Go to the MainDemo.Web and change the debug properties as follows:

{% img /images/blog/xaf-with-glimpse-015.png %}

Then run the web application. Glimpse is off by default. In order to turn it on, you need to set a cookie, which you can do by navigating to `Glimpse.axd`. This is the Glimpse configuration screen.

{% img /images/blog/xaf-with-glimpse-014.png %}

## Additional Glimpse extensions ##

Glimpse is extensible and there are [many additional plugins](http://getglimpse.com/Packages) available. Many of these take the form of additional tabs with details about a particular aspect of your application: SQL Queries, Elmah errors, ASP.NET caching, logging, dependency injection, _etc._

## MainDemo Sample Project ##

I have uploaded a modified XAF MainDemo to this [GitHub repository](https://github.com/ZeroSharp/Xaf_MainDemo_Glimpse) with all the above changes. (Don't forget to debug against IIS Express).