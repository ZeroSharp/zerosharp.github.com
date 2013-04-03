---
layout: post
title: "Load Testing XAF: Part 5 - Analysis"
date: 2013-04-14 07:37
comments: true
categories: [c#, devexpress, xaf, aws, neustar]
description: Analysis of the load test we ran against the DevExpress XAF MainDemo.
published: false
---
This is the final post in a series about load testing XAF applications.  Previously in the series: 

* [Load Testing XAF: Overview](/load-testing-xaf-overview/)
* [Part 1: Deploying the target webserver](/load-testing-xaf-part-1-deploying/)
* [Part 2: Selenium](/load-testing-xaf-part-2-selenium/)
* [Part 3: Uploading and validating a script](/load-testing-xaf-part-3-uploading-and-validating-the-virtual-user-script/)
* [Part 4: Launching the load test](/load-testing-xaf-part-3-uploading-and-validating-the-virtual-user-script/)

In this part, we analyse the results of the load test we ran in [Part 4](/load-testing-xaf-part-3-uploading-and-validating-the-virtual-user-script/).

## Results ##

The results of the test we ran are [here]( 
https://load.wpm.neustar.biz/load/test/share/e0f547d711624168bc7fa0a69ddd8283). The graphs are interactive and give quite interesting data about the load tests. Feel free to have a look and play around with the results.

## The Performance Graphs ##

{% img /images/blog/load-testing/load-testing-006.png %}

The above graph shows the basic information about the test. We can see there were 649 transactions (the Selenium script was run 649 times) and there were 17 errors.

You can see that the test managed to follow the planned scenario (the actual number of users follows the yellow line). You can also see information about the throughput in bytes during the test.

## The Script Performance Graphs ##

Here we see information pertaining to the script we chose to run. Had we run multiple different scripts, we would be able to isolate each one.

{% img /images/blog/load-testing/load-testing-009.png %}

It is clear that the transaction time rises quite slowly with the load until about 12:17 when there is a jump. More on that later.

First, I have removed the plot of the total transaction time, so that we can see more detail from the remaining steps. Two things seem to be clear: the response time for each individual step does not seem to be much affected by the load, but the login step rises gradually.

{% img /images/blog/load-testing/load-testing-010.png %}

Now back to the spike at 12:17, if you look at the transaction data more closely (not shown here but available on the [NeuStar results page](https://load.wpm.neustar.biz/load/test/share/e0f547d711624168bc7fa0a69ddd8283), it looks like several transactions finished at the same time and that the jump coincides with several simultaneous logins.

So again, it seems that login is the 'weakest link', i.e., the most resource intensive step and the one that suffers the most under load.

## The Errors ##

Lets look more closely at the errors.

{% img /images/blog/load-testing/load-testing-007.png %}

The first type of error we can see from the screenshot occurred at the login page. This error happened 6 times and was very similar to another error which occurred once. In fact, all 6 of these errors happened at the very end of the test. As such, they can be ignored, because it is likely that the load test was scaling down and interrupting sessions at this point.

{% img /images/blog/load-testing/load-testing-008.png %}

One of the very nice features of NeuStar's load testing solution is that you not only get a screenshot, but also a video which shows you exactly what the user experienced when an error occurs. By clicking on the second error, it looks like there may be a problem  when the system is under load. There were 6 errors and it is clear in the video that the errors occur when attempting to navigate away from the _Scheduler Event_ view to the _My Details_ view. The screenshot gives us some useful information. DevExpress? Any ideas?

{% img /images/blog/load-testing/load-testing-011.jpg %}

There are a couple of other errors, but I think these are most likely a result of problems with the Selenium script rather than XAF. The AJAX update panels make it quite hard to detect when the page is fully ready and although we try to accommodate this with `selenium.waitForCondition()` my feeling is that the `click()` occurred before everything was properly wired up.

## Further tests ##

This series has covered the process of load testing XAF applications in its entirety, but in some ways, it feels like only the beginning. There are many other configurations and tests which would be interesting.

For instance

* Reduce think time which is currently set to 3 seconds per step
* Use a smaller/larger EC2 instance
* Increase the number of virtual users
* Try load balancing with sticky sessions
* Experiment with/without compression (IIS or via the `web.config`)
* Experiment with/without caching (both at the http level, and via the [cached data store](http://documentation.devexpress.com/#XPO/CustomDocument9892)

## A note about concurrency ##

In our experience, people tend to over estimate the number of concurrent users for their application. Our application has probably upwards of 4000 users defined, but we know from our logs that there have never been more than 80 simultaneously logged in. Also, even with 80 concurrent users, they have a much slower 'think time' than 3 seconds on average.

For our production environment, we run at least one 25 user test for every major release and ensure the performance is at least as good as the previous release. In production, the system is load balanced (with sticky sessions) and we know from experience that this is sufficient.
