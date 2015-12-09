---
layout: post
title: "Migrating a large web application from XAF 12.1 to 15.1 - Part 3"
date: 2015-09-16 09:39
comments: true
categories: [c#, devexpress, xaf]
description: Describing some of the challenges encountered in upgrading to the latest DevExpress expressAppFramework. Success - and some satisfying results from the multi-user load test.
---
{% pullquote %}
This is the third part of a [series](/migrating-a-large-web-application-from-xaf-12-dot-1-to-15-dot-1-part-2) about migrating a large application from XAF 12.1 to XAF 15.1. In this part I will compare the results of a simple stress test between the versions.

I have described in previous posts how to [stress test XAF applications](/load-testing-xaf-overview/). One of our most basic tests is to simulate 25 users cycling through all the navigation tabs for an hour. {"I'm happy to report there is a considerable improvement under load in version 15.1."}

(Note that we purposefully stress test against a single web application so that we can compare apples with apples. In production we have multiple instances load-balanced.)

Here is an interactive summary of the 15.1 results:
{% endpullquote %}

Here is version 15.1. There were zero errors and 382 completed scripts. 

By comparison, the same test against DevExpress 12.1 yielded only 258 completions. So 15.1 shows a 48% performance improvement over 12.1.

{% iframe cmd https://load.wpm.neustar.biz/load/test/share/e4c5e109f8f04ceebd72cdb5c93eb1c2 1024 512 %}



