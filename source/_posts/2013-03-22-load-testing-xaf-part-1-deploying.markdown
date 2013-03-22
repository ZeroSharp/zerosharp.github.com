---
layout: post
title: "Load Testing XAF: Part 1 - Deploying"
date: 2013-03-22 11:21
comments: true
categories: [c#, devexpress, xaf, aws, neustar]
description: How to deploy the XAF MainDemo ready for load testing.
---
This is the first part of a tutorial about load testing XAF applications. See the [overview](/load-testing-xaf-overview/) for a bit of background. In this post we set up the target webserver.

You can target any machine which has a publicly available web address, but for this tutorial, I'm' deploying the MainDemo to the Amazon cloud, by following the instructions in [Part 1](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-1-putting-the-database-in-the-cloud/) and [Part 2](http://blog.zerosharp.com/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-2-publishing-maindemo/) of my previous series about Amazon Web Services.

I am using version 12.2.7 of the DevExpress XAF MainDemo. There are a couple of extra changes to make to the web.config.

* Set debug to false `<compilation targetFramework="4.0" debug="false">` in the `<compilation>` section of `<system.web>`
* Switch to _Release_ mode before deploying.

There are a couple of differences compared to the tutorial:

* I chose a _Medium_ instance instead of a _Micro_ instance for EC2 (the web server) and deployed it against IIS 8.
* For RDS (the database), I stuck with a _Micro_ instance.

{% img /images/blog/load-testing/load-testing-001.png %}

For the load test, it is also important to disable the automatic health checks performed by the load balancer. 

{% img /images/blog/load-testing/load-testing-002.png %}

The reason for this is that we are trying to determine the breaking point of our application. If the elastic load balancer detects that a system is struggling, it might automatically flag it as unhealthy and replace it with a newly launched instance. While this behaviour might be desirable for a production system, it doesn't make sense for a load test.

Make sure you can connect to your installation from a web browser before continuing. I chose to deploy to a Windows 2012 instance running IIS 8.0 (which was not available when I wrote my previous XAF AWS tutorial) and I had a little trouble with the URL. If I navigate to the application's base URL (_http://zerosharp-maindemo.elasticbeanstalk.com/_ in my case), then I get forwarding problems after logging in. Instead, I navigate to the full address _http://zerosharp-maindemo.elasticbeanstalk.com/MainDemo.Web_deploy/Default.aspx_ and everything works. I'll try to look into it later, but it's not important for the load testing.