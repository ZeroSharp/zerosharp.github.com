---
layout: post
title: "Load Testing XAF: Bonus - Simultaneous EasyTests"
date: 2013-04-22 16:31
comments: true
categories: [c#, devexpress, xaf, easytest]
description: Local load testing of the DevExpress XAF MainDemo using simultaenous EasyTests.
---
In [my recent series on load testing XAF](/load-testing-xaf-overview), I used a Selenium javascript test to run the client browser instances. This is a good and cheap method of validating the performance of XAF applications under production load.

However, if the load tests fail because of a concurrency bug or a performance bottleneck, it can still be difficult to analyse and solve. For this, we need to be able to simulate load locally against the development environment.

In this post I will demonstrate how to run multiple simultaneous XAF EasyTests against a local server. As a load test, it is not very scientific, but it can be extremely useful as a debugging tool.

## The EasyTest script ##

First, we will create a new EasyTest which will cycle through the existing navigation tabs. Open the XAF MainDemo and create a new EasyTest as follows.

{% codeblock MainDemo_CycleThroughTabs.ets lang:text %}
#Application MainDemoWeb

*FillForm
 User Name = Sam
 Password = 
*Action Log On

*Action Navigation(Contact)
*Action Navigation(Task)
*Action Navigation(Department)
*Action Navigation(Scheduler Event)
*Action Navigation(My Details)
*Action Navigation(Note)
*Action Navigation(Payment)
*Action Navigation(Position)
*Action Navigation(Resume)
*Action Navigation(Role)
*Action Navigation(User)
*Action Navigation(Reports.Analysis)
*Action Navigation(Reports.Reports)
*Action Log Off
{% endcodeblock %}

(This test replicates the Selenium test we created in [Part 2](/load-testing-xaf-part-2-selenium/) of my previous series on load testing with NeuStar and Amazon.) It is important to note that we are only testing the web application and that we do not include a `#DropDB` directive.

First, ensure that you can run this test with the default settings.

## The config file ##

Now modify the config.xml file as follows:

{% codeblock Config.xml lang:xml %}
<?xml version="1.0" encoding="utf-8" ?>
<Options xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" TestRunTimeLimit="5" >
  <Applications>
    <!-- Web -->
    <Application
      Name="MainDemoWeb"
      Url="http://localhost:4030"
      SingleWebDev="True"
      WebBrowserType="Standalone"
      PhysicalPath="[ConfigPath]\..\MainDemo.Web"
      AdapterAssemblyName="DevExpress.ExpressApp.EasyTest.WebAdapter.v12.2, Version=12.2.8.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a"/>
    <!-- For IIS -->
    <!--<Application
      Name="MainDemoWeb"
      Url="http://localhost/MainDemo.Web/Default.aspx"
      PhysicalPath=""
      DontRestartIIS="True"
      DontRunWebDev="True"
      WebBrowserType="Standalone"      
      AdapterAssemblyName="DevExpress.ExpressApp.EasyTest.WebAdapter.v12.2, Version=12.2.8.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a"/-->
  </Applications>
</Options>
{% endcodeblock %}

There are a few important things to note. 

I have not shown the _Win_ section here since we are not using it. Also, I am using XAF 12.2.8. You may need to change the version number in the `AdapterAssemblyName` attribute. I have increased the `TestRunTimeLimit` attribute from 3 to 5. Everything goes a little slower when there are multiple browsers and we need to make sure the test does not time out.

With the above config, the EasyTest will no longer run from within Visual Studio.

You can choose to run the simultaneous tests against the debug web server or IIS. Uncomment the relevant section. The interesting settings are:
 
 - `SingleWebDev="True"` which instructs the EasyTest runner to run all tests against the same instance of the development webserver. Without this, the webserver would be stopped and started for each test.
 - `WebBrowserType="Standalone"` which causes each launched browser to be launched with its own session. (There are a few mentions of this setting in the support center, but it is not very well documented).
 - `DontRestartIIS` and `DontRunWebDev` which are self-explanatory

## The launch command ##

Next, create the following batch file in the MainDemo.EasyTests subdirectory. ##

{% codeblock Launch.bat lang:bat %}
:: Requires the Debug webserver to be running on port 49660
:: Requires EasyTests to be enabled
:: Requires NetDA to be running
:: Requires admin rights
:: Must be run from a command prompt
::
:: Usage: > launch <numberOfBrowsers>
:: e.g. : > launch 21
:: will launch 21 simultaneous browsers at 3 second intervals

@echo off

:DELETE_OUTPUT
if exist *.jpeg del *.jpeg
if exist *.html del *.html
if exist TestsLog.xml del TestsLog.xml

:CHECK_ADMIN
net session >nul 2>&1
if %ERRORLEVEL% equ 0 goto CHECK_CONSOLE
echo Must be run from an administrative command window
goto ERROR

:CHECK_CONSOLE
echo %CMDCMDLINE% | find /i "/c" >nul
if ERRORLEVEL 1 goto CHECK_PARAMS
echo Must be run from an administrative console (not Windows Explorer)
goto ERROR

:CHECK_PARAMS
IF [%1]==[] GOTO USAGE

:LAUNCH
set /a i=0

:LOOP
if %i%==%1 goto OK
set /a i=%i%+1
start "x" "C:\Program Files (x86)\DevExpress\DXperience 12.2\Tools\eXpressAppFramework\EasyTest\TestExecutor.v12.2.exe" MainDemo_CycleThroughTabs.ets
:: Wait 3 seconds
ping 1.1.1.1 -n 1 -w 3000 >nul
goto LOOP

:USAGE
echo Usage: %0 numberOfBrowsers
echo numberOfBrowsers must be an integer
goto OK

:ERROR

:OK
pause
{% endcodeblock %}

If you want to run your tests against the development webserver, you will need to make sure it is running before launching the batch file. The easiest way to do this is to run the application from within Visual Studio and then close the browser. You should still see the development webserver running in the task bar notification area. Against IIS, it is enough to ensure it is started.

Now, open an administrative command prompt. Note that you must run from an administrative console: it is not sufficient to 'run as administrator' from Windows Explorer. Navigate to the EasyTest subdirectory where the Launch.bat file is located and launch a single test with the following command:

    launch.bat 1

You should see the test run without error. If this works, you can then launch 20 simultaneous test runs with 3 second intervals by running:

    launch.bat 20

## Conclusion ##

As a load test, you do not get much useful information. Even if we managed to extract accurate data for client response times and throughput, the overhead of running the multiple browsers would skew the results too much. However, this approach is extremely useful for isolating concurrency and performance problems.


