---
layout: post
title: "Load Testing XAF: Part 4 - Launching the load test"
date: 2013-04-08 11:19
comments: true
categories: [c#, devexpress, xaf, aws, neustar]
description: How to launch a load test using NeuStar Web Performance Management.
---
This is another post in a series about load testing XAF applications.  Previously in the series: 

* [Load Testing XAF: Overview](/load-testing-xaf-overview/)
* [Part 1: Deploying the target webserver](/load-testing-xaf-part-1-deploying/)
* [Part 2: Selenium](/load-testing-xaf-part-2-selenium/)
* [Part 3: Uploading and validating a script](/load-testing-xaf-part-3-uploading-and-validating-the-virtual-user-script/)

In this part, we will launch a 1 hour test with 25 virtual users using the [NeuStar Web Performance Management](http://home.wpm.neustar.biz/) module.

## Schedule and launch a test ##

From the script validation screen, click on _Schedule a load test with this script_. The defaults are good, but you can specify in detail how to run your load test. For instance, you can coordinate multiple Selenium scripts to simulate different types of activity on your site.

{% img /images/blog/load-testing/load-testing-005.png %}

Notice that the load test cost for 25 users for an hour will be only $3.75.

When you click `Launch`, Neustar takes 7 or 8 minutes to provision the Amazon machines and stage the test, after which you will get realtime detail information about response times, bandwidth and errors. 

In the next post we'll analyse the results of this test.

