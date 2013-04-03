---
layout: post
title: "Load Testing XAF: Part 3 - Uploading and Validating the Virtual User Script"
date: 2013-04-03 16:39
comments: true
categories: [c#, devexpress, xaf, aws, neustar]
description: How to upload and validate a script using NeuStar Web Performance Management.
---
This is another post in a series about load testing XAF applications.  Previously in the series: 

* [Load Testing XAF: Overview](/load-testing-xaf-overview/)
* [Part 1: Deploying the target webserver](/load-testing-xaf-part-1-deploying/)
* [Part 2: Selenium](/load-testing-xaf-part-2-selenium/)

In this part, we will load test the application we set up in [Part 1](/load-testing-xaf-part-1-deploying/), using the Selenium load test we created in [Part 2](/load-testing-xaf-part-2-selenium/).

## Neustar Web Performance Management ##

NeuStar (formerly BrowserMob) are a company specialised in web application performance monitoring. We are interested in their [web performance module](https://home.wpm.neustar.biz/). It is free to create an account. To run a test with less than 25 virtual users costs only $0.15 per virtual user. Tests with more than 25 users (up to 5000) require an additional paid plan.

## Create a script ##

In order to run a load test, we first need to create the script and validate it. Go to the [scripting](https://script.wpm.neustar.biz/) page and select 'Create a new script'. Then cut and paste the [Selenium code for `MainDemo_CycleThroughTabs.js`](https://gist.github.com/shamp00/5302223) from the [previous post](/load-testing-xaf-part-2-selenium/).

Now change the `targetHost` variable near the top of the file to point to the location of your MainDemo installation. You can then validate the script. This will actually run through the Selenium test on a newly provisioned Amazon instance to ensure that it passes.

{% img /images/blog/load-testing/load-testing-004.png %}

If you get a green icon, you can proceed with setting up a load test, otherwise you can see what went wrong in a video of the user session.

In the next post we will configure and launch the load test.
