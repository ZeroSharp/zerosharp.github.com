---
layout: post
title: "Load Testing XAF: Overview"
date: 2013-03-12 18:20
comments: true
categories: [c#, devexpress, xaf, aws, neustar]
description: An overview of a new series of posts about load testing an ASP.NET application based on DevExpress XAF.
---

Over the next few posts, I will demonstrate how to load test XAF web applications.

## History ##

Performance testing has traditionally been difficult and expensive. A few years ago, to do it well required a powerful piece of dedicated load testing software such as HP LoadRunner (typical cost back in 2007: USD 50,000-100,000 or more per year!). This software was capable of simulating multiple virtual users via the use of recorded scripts and providing detailed performance statistics. Usually the cost was increased further increased by the need for powerful hardware to be able to run the application.

In 2007 we were required by a big customer (a global bank) to provide load testing statistics for our expressApp Framework application. We could not afford anything as sophisticated as LoadRunner so we went with a cheaper alternative (NeoLoad, which was still several thousand per year). It was extremely painstaking work to produce a test. The approach was to record the http requests (as a stream of text values) and write a script to check the http response. Since there was no real browser involved, it was very difficult to determine if our test was really representative. Several machines in the office had to be dedicated to simulating virtual users. Nobody was allowed to use the internet for fear of skewing the results during a test. If an error occurred, it was almost impossible to determine what went wrong. We wrestled with it and managed to fulfill our requirements, but it was all a lot of effort for no real return.

Part of the problem was certainly that XAF is complex to test. The user interface is rich and makes use of complex controls. Most of these load testing tools work better when targetting a simple `<INPUT type="button">` rather than an image of a button that sometimes is not even clickable until the mouse has hovered over it. DevExpress have made it easy to run tests via their EasyTests, but no load testing tool supports them yet. (They have informed me it's in their plans...)

## Enter the cloud ##

{% pullquote %}

The basic idea is this: instead of simulating users with specialised software, why not fire up a virtual machine and test with a real browser instance which is 'remote controlled' via a script.

{"The increased availability of cheap cloud-based virtualised machines has revolutionised load testing."} The rental of the virtualised machines is not free, but it is very cheap.  In about 2008, I started using the Amazon cloud to perform load tests. Our basic test costs us about USD 10.00 per run. We probably run this a dozen times a year, so our total cost is about USD 120.00 per year.
{% endpullquote %}

We get better statistics than we ever got out of NeoLoad. We are confident that the test is realistic and we can compare with the actual performance of our production environments. We have been able to find and solve memory leak problems and various tricky multi-user problems with these tests.

Load testing is still a complex business. There are a lot of pieces to put together, but with the cloud, each piece is relatively simple and cheap.

## The solution ##

* In [Part 1](/load-testing-xaf-part-1-deploying/), we install the DevExpress MainDemo on an Amazon EC2 instance.
* In [Part 2](/load-testing-xaf-part-2-selenium/), we use Selenium to write a script which will run on the client machines to control a real browser instance.
* In [Part 3](/load-testing-xaf-part-3-uploading-and-validating-the-virtual-user-script/), we create a NeuStar Web Performance test and validate the script.
* In [Part 4](/load-testing-xaf-part-4-launching-the-load-test/), we launch a performance test and monitor the server.
* In [Part 5](/load-testing-xaf-part-5-analysis/), we analyse the results of the clients.

If you follow all the steps, expect to pay a handful of dollars in Amazon EC2 costs and a few more in Neustar costs. You could alternatively skip step 1 and target one of your own development machines instead.