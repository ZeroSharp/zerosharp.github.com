---
layout: post
title: "DevExpress 14.2 review"
date: 2015-01-20 08:35
comments: true
categories: [c#, devexpress, xaf]
description: A review of the new DevExpress 14.2 release with particular focus on the XAF ASP.NET report designer.
---
{% pullquote %}
This post is an overview of the brand new version XAF 14.2. {"The truly outstanding new feature is the ASP.NET report writer"} which is now available in all XAF applications.

A few months ago, we lost a potential sale because the customer wanted the ability to create custom reports from within the browser. We told them it was impossible to provide a fully-fledged report designer within our web application - but the DevExpress guys have done it! And how! {% endpullquote %}

## The web-based report designer ##

Let's fire up the MainDemo application and navigate to the reports view. The first thing to notice is that there is a new action _Show Report Designer_. 

{% img /images/blog/devexpress-14-2-review-001.png %}

The designer action is disabled because the selected report is _predefined_. Predefined reports are a feature of Reports v2 which were introduced in version 13.2 ([see my previous review](/devexpress-13-dot-2-review-part-1)). So first, we clone the existing predefined report. I renamed the copy (via the edit button) so that we can tell them apart.

{% img /images/blog/devexpress-14-2-review-002.png %}

Now the _Show Report Designer_ action is enabled. Let's click it. _Whoa! That's one impressive user interface for a web application!_

{% img /images/blog/devexpress-14-2-review-003.png %}

Let's add a chart and a few controls. I thought (incorrectly) that the link to the domain model might be somewhat lacking because the report designer is not designed specifically for XAF (you can also use it with non-XAF ASP.NET or ASP.NET MVC applications) but navigating the available domain objects to select a property seemed very natural and simple.

{% img /images/blog/devexpress-14-2-review-005.png %}

I had a few little mouse issues while trying to resize or move controls, and there were a couple of places where the interface seemed slightly sluggish, but these were very minor issues. In general the designer is slick and easy to use. I also had a little difficulty finding the _Save_ button, but here it is:

{% img /images/blog/devexpress-14-2-review-007.png %}

And here's the live output after my modifications.

{% img /images/blog/devexpress-14-2-review-006.png %}

You can also start from scratch with a new blank report.

{% img /images/blog/devexpress-14-2-review-004.png %}

This report designer is an **extremely impressive achievement**. I played around with it for over an hour and it did not crash once. I managed to implement everything I tried including a chart, a bar code and a new data field. 

There are some features missing from the web-based report designer compared to the Windows Forms version. Most significant is the ability to attach events and scripts to controls. Here is a [full feature comparison table](https://documentation.devexpress.com/#XtraReports/CustomDocument14651).

I had a quick look for the tools they used to implement it. It looks like it uses [jQuery](https://jquery.com), [jQuery.UI](https://jqueryui.com) and [knockout.js](https://knockoutjs.com) and you can automatically bundle the required libraries via a new setting in the web.config. There is [some more information here](https://documentation.devexpress.com/#XtraReports/CustomDocument17558).

On the whole I am utterly impressed. Hats off to the DevExpress team!

## Other new features in XAF 14.2 ##

The new 14.2 includes several other new features. These include the ability to store user settings in the data store as well as improvements to the speed of the grids. For a full list of the new features and improvements see [here](https://community.devexpress.com/blogs/eaf/archive/2014/11/18/xaf-brand-new-module-amp-features-for-both-windows-and-the-web-coming-soon-in-v14-2.aspx) and [here](https://community.devexpress.com/blogs/eaf/archive/2014/11/20/xaf-enhancements-to-existing-features-amp-performance-tuning-coming-soon-in-v14-2.aspx).
