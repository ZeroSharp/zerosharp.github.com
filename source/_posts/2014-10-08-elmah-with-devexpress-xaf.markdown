---
layout: post
title: "ELMAH with DevExpress XAF"
date: 2014-10-08 15:45
comments: true
categories: [c#, devexpress, xaf, logging]
description: How to add ELMAH (Error Logging Modules and Handlers) to the XAF MainDemo web application.
---
[ELMAH (Error Logging Modules and Handlers)](https://code.google.com/p/elmah/) is an open source library for logging unhandled exceptions. This post explains how to get it running with the [DevExpress XAF](https://www.devexpress.com/Products/NET/Application_Framework/) main demo.

A couple of amazing facts about ELMAH. 

* It has been around since 2004! 
* It was written by [Atif Aziz](http://www.raboof.com/) who happens to be an old school-friend from the International School of Geneva.

XAF provides [quite extensive error handling options](https://documentation.devexpress.com/#xaf/CustomDocument2704) out of the box, but I have found Elmah better suited to production environments because of the ability to remotely view the full error log.

## Setting up ##

First, get the ELMAH package via NuGet into the MainDemo.Web project. ELMAH provides dozens of different methods of persisting the error log. For this example we'll choose one of the simplest. Make sure you select the _ELMAH on XML Log_ package.

{% img /images/blog/xaf-with-elmah-001.png %}

NuGet makes several automatic modifications to the _web.config_. Unfortunately, these are not quite accurate enough for XAF. The changes you need to make are detailed below:

Add a `<configSection>` for ELMAH as alongside the existing devExpress one.

```xml web.config
  <configSections>
    <sectionGroup name="devExpress">...</sectionGroup> <!-- this should already exist-->
    <sectionGroup name="elmah"> <!-- this is new-->
      <section name="security" requirePermission="false" type="Elmah.SecuritySectionHandler, Elmah" />
      <section name="errorLog" requirePermission="false" type="Elmah.ErrorLogSectionHandler, Elmah" />
      <section name="errorMail" requirePermission="false" type="Elmah.ErrorMailSectionHandler, Elmah" />
      <section name="errorFilter" requirePermission="false" type="Elmah.ErrorFilterSectionHandler, Elmah" />
     </sectionGroup>
  </configSections>
```

Your `<system.webServer>` section should look like this:

```xml web.config
  <system.webServer>
    <handlers>...</handlers> <!-- This is unchanged -->
    <validation validateIntegratedModeConfiguration="false" />
    <modules>
      <add name="ASPxHttpHandlerModule" type="DevExpress.Web.ASPxClasses.ASPxHttpHandlerModule, DevExpress.Web.v14.1, Version=14.1.7.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
      <add name="ErrorLog" type="Elmah.ErrorLogModule, Elmah" preCondition="managedHandler" />
      <add name="ErrorMail" type="Elmah.ErrorMailModule, Elmah" preCondition="managedHandler" />
      <add name="ErrorFilter" type="Elmah.ErrorFilterModule, Elmah" preCondition="managedHandler" />
    </modules>
  </system.webServer>
```

Add a `<location>` for the path _elmah.axd_ (alongside the existing `<location>` tags).

```xml web.config
  <location path="elmah.axd" inheritInChildApplications="false">
    <system.web>
      <httpHandlers>
        <add verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" />
      </httpHandlers>
      <!-- 
        See http://code.google.com/p/elmah/wiki/SecuringErrorLogPages for 
        more information on using ASP.NET authorization securing ELMAH.

      <authorization>
        <allow roles="admin" />
        <deny users="*" />  
      </authorization>
      -->  
    </system.web>
    <system.webServer>
      <handlers>
        <add name="ELMAH" verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" preCondition="integratedMode" />
      </handlers>
    </system.webServer>
  </location>
```

Add a new `<elmah>` section. I put mine just before the final `</configuration>` tag.

```xml web.config
  <elmah>
    <errorLog type="Elmah.XmlFileErrorLog, Elmah" logPath="~/App_Data/Elmah.Errors" />
    <!--
        See http://code.google.com/p/elmah/wiki/SecuringErrorLogPages for 
        more information on remote access and securing ELMAH.
    -->
    <security allowRemoteAccess="false" />
  </elmah>
```

Now modify _HttpModules.Web.Config_ to look like this:

```xml HttpModules.Web.Config
<?xml version="1.0" encoding="utf-8"?>
<httpModules>
  <add name="ASPxHttpHandlerModule" type="DevExpress.Web.ASPxClasses.ASPxHttpHandlerModule, DevExpress.Web.v14.1, Version=14.1.7.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a" />
  <add name="ErrorLog" type="Elmah.ErrorLogModule, Elmah" />
  <add name="ErrorMail" type="Elmah.ErrorMailModule, Elmah" />
  <add name="ErrorFilter" type="Elmah.ErrorFilterModule, Elmah" />
</httpModules>
```

Now we need to extend XAF's standard error handling. Create a new class in the web application.

```c#
public class ElmahErrorHandling : ErrorHandling
{
    protected override void LogException(ErrorInfo errorInfo)
    {
        base.LogException(errorInfo);

        if (errorInfo.Exception != null)
            Elmah.ErrorSignal.FromCurrentContext().Raise(errorInfo.Exception);
    }
}
```

And then modify _Global.asax.cs_ to instantiate the new class

```c#
        protected void Application_Start(object sender, EventArgs e) {
            ErrorHandling.Instance = new ElmahErrorHandling(); // <---this line is new
            ASPxWebControl.CallbackError += new EventHandler(Application_Error);
#if DEBUG
            TestScriptsManager.EasyTestEnabled = true;
#endif
        }
```

The complete files are available with the [source code](https://github.com/ZeroSharp/Xaf_MainDemo_Elmah).

Now run the application and trigger an unhandled exception. Change the URL to something that does not exist. Or open any detail view and modify the URL so that the Guid in the _ShortcutObjectKey_ is invalid (replace a digit with an 'X'). Then the application error page appears.

{% img /images/blog/xaf-with-elmah-002.png %}

Then return to the application and change the URL to `Elmah.axd`. You are looking at the log of all unhandled exceptions.

{% img /images/blog/xaf-with-elmah-003.png %}

And for each exception, you can view the full details of any logged exception including coloured stack trace and full server variables.

{% img /images/blog/xaf-with-elmah-004.png %}

## ELMAH options ##

By default, ELMAH is configured to disallow remote access to the error logs -  only a local user can get to _elmah.axd_. If you take care of the security implications it can be very useful to enable remote access and  monitor the logs on your production servers.

We chose to use an XML file for each error but ELMAH is entirely pluggable. There are dozens of alternatives for persisting the error log including Sql Server, an RSS feeds, to Twitter, even to [an iPhone app](http://code.google.com/p/elmah/wiki/ProwlingErrors). There are even third party sites such as [elmah.io](http://elmah.io) who will host your error logs for you.

One of the advantages of using XML files is that the files can be copied to another machine. If you look in *MainDemo.Web\App_Data\Elmah.Errors*, you will find the resulting xml files.

{% img /images/blog/xaf-with-elmah-005.png %}

You can just copy these files to another installation's *Elmah.Errors* folder and the log will show up when you visit _Elmah.axd_.

One final note. ELMAH was developed for ASP.NET applications and web services, but it is possible to get it to work with other types of applications such as Windows Forms, Windows Service or console applications. Check out [this StackOverflow question](https://stackoverflow.com/questions/841451/using-elmah-in-a-console-application).

The source code for this example is [on GitHub](https://github.com/ZeroSharp/Xaf_MainDemo_Elmah).