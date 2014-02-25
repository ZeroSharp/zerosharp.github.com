---
layout: post
title: "CruiseControl Notifications on your iPhone"
date: 2014-02-07 11:32
comments: true
categories: [ccnet, ios, iphone]
description: A review of the CCWatcher iOS app from SixAfter for monitoring your build server.
---
{% img right /images/blog/ccwatcher-002.jpg 200 %}

[CCWatcher](http://sixafter.com/portfolio/continuous-integration-monitoring-for-ios/) is a great iPhone app for any developer who is using CruiseControl.NET, Jenkins or Hudson for their continuous integration.

We have used [CruiseControl.NET](http://www.cruisecontrolnet.org/) for several years to automate all builds. The build kicks off automatically whenever we push changes and this frequently happens a few times a day. We aim to have everything _green_ at the end of every day. 

A full build with unit tests and functional tests takes about an hour, so often, I will leave the office while the build is still running. But then I wouldn't know if I'd broken the build.

For a long while I was looking for a better way of being informed of build failures on my phone.

{% img left /images/blog/ccwatcher-001.png 300 %}

Enter **CCWatcher**. It's a simple application which allows you to enter the address of your build server and it will tell you the status of the projects you've configured.

Whenever I need to check whether I've broken the build, I can just _pull to refresh_.

When I first discovered the application it had a problem with CruiseControl.NET's [remote services](http://cruisecontrolnet.org/projects/ccnet/wiki/RemoteServices) (which we needed because the actual build server is on a network machine not visible from the internet). I contacted _SixAfter_ and was very impressed with the helpful response from Michael Primeaux. Within a couple of weeks there was a new version of the app with support for the remote services configuration. Hats off.