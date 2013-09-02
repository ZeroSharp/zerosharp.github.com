---
layout: post
title: "Always run Visual Studio as an Administrator"
date: 2013-09-09 10:49
comments: true
categories: [visualstudio]
---
I always run Visual Studio as an administrator. There are various reasons why this is necessary including:

* using IIS with a web application
* running web UI tests
* profiling

In fact there's a great list [on MSDN](http://msdn.microsoft.com/en-us/library/vstudio/jj662724.aspx) of all the actions you require administrator permissions.

There is a way to make sure Visual Studio always opens with elevated privileges, even if you double click on a _.sln_ file. (I'm running Windows 8 - not sure if it works with other versions of Windows.)

* Right-click _devenv.exe_
* Select _Troubleshoot program_
* Check _The program requires additional permissions_
* Click _Next_ 
* Click _Test the program..._
* Wait for the program to launch
* Click _Next_
* Select _Yes, save these settings for this program_
* Click _Close_

If you have multiple versions of Visual Studio installed, you should repeat the operation for all of the following.

Program | &nbsp;Default location
--- | ---
&nbsp;Visual Studio 2008&nbsp; | &nbsp;_C:\Program Files (x86)\Microsoft Visual Studio 9.0\Common7\IDE\devenv.exe_ &nbsp;
&nbsp;Visual Studio 2010&nbsp; | &nbsp;_C:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\IDE\devenv.exe_ &nbsp; 
&nbsp;Visual Studio 2012&nbsp; | &nbsp;_C:\Program Files (x86)\Microsoft Visual Studio 11.0\Common7\IDE\devenv.exe_ &nbsp;
&nbsp;VSLauncher.exe&nbsp; | &nbsp;_C:\Program Files (x86)\Common Files\Microsoft Shared\MSEnv\VSLauncher.exe_ &nbsp;

&nbsp;