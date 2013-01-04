---
layout: post
title: "Deploy XAF ASP.NET Applications to Amazon Web Services: Part 5"
date: 2012-12-05 11:03
comments: true
categories: [c#, devexpress, xaf, aws]
description: A tutorial for publishing the DevExpress expressAppFramework MainDemo to Amazon Web Services. This part covers load balancing.
---

## Part 5: Load balancing ##

This is the final post in this 5 part series about deploying XAF to the Amazon cloud. The other parts are:

* [Part 1 - Creating a database in RDS](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-1-putting-the-database-in-the-cloud/)
* [Part 2 - Deploying to ElasticBeanstalk](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-2-publishing-maindemo/)
* [Part 3 - Troubleshooting](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-3-troubleshooting-via-remote-desktop/)
* [Part 4 - Incremental Deployment](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-4-incremental-deployment/)

This part covers load balancing and autoscaling.

### Load balancing ###

In order to enable load balancing with DevExpress XAF applications, you must activate client affinity (sticky sessions) which you can do in the **Load Balancer tab**.

{% img /images/blog/aws/aws-autoscaling-2.jpg 'figure 19' %}

This is because XAF relies http session variables (notably the `WebApplication`). With sticky sessions enabled, when a user first hits the Elastic Load Balancer, they are assigned a random server but all subsequent requests are routed to the same server instance. 

## Autoscaling ##

### Adding Capacity (Works) ###

Switch to the **Auto Scaling** tab of the Elastic Beanstalk instance.

{% img /images/blog/aws/aws-autoscaling-1.jpg 'figure 19' %}

The options for autoscaling are extensive and there is plenty information on the [Amazon website](http://aws.amazon.com/autoscaling). In the above screenshot we have the Elastic Load Balancer configured so that it will initially deploy 1 server (the Minimum Instance Count), but whenever the NetworkOut measurement of 600,000 bytes over a 5 minute interval, a new server launch is triggered up to a maximum of 4 servers. Adding capacity works fine with XAF applications.

### Removing Capacity (Fails) ###

The configuration above also shows that if the NetworkOut drops to below 200,000 bytes over a period of 5 minutes then the number of servers should decrease. **This will NOT work**.

The problem is that the Elastic Load Balancer will terminate the surplus server immediately (the equivalent of a `shutdown -s`). Any users connected to that server will immediately lose their session and will probably receive an ugly error.

One workaround is to allow scaling out but to disable scaling in. To do this, set the 'Lower Threshold' to zero.

### Possible Improvements ###

There are a couple of potential longer term solutions to the problem of dropped sessions when scaling in.

DevExpress could make the classes they use in the session variables serializable (e.g., `WebApplication`). When this is the case, the application can be configured to store the session variables to a persistent data store (SQL Server for instance) instead of in the session and then we could configure autoscaling to work correctly. There is [an item in the Support Center in this regard](http://www.devexpress.com/Support/Center/Question/Details/Q426045) and [another one here](http://www.devexpress.com/Support/Center/p/S36497.aspx).

Another solution would be if Amazon provided a new option to handle a server termination in a less brutal fashion. That is, stop routing new users to the surplus server but continue to server existing users until their sessions have all terminated. This approach is discussed in [this AWS forum thread](https://forums.aws.amazon.com/thread.jspa?threadID=61278).

## Conclusions ##

The load balancing features of AWS work fine with XAF applications, but the benefits of autoscaling are lost because there is currently no effective method of removing capacity without knocking out existing sessions.

## What next ##

In a future series, I will demonstrate how to load test XAF applications cheaply and effectively by making use of Amazon Web Services. Stay tuned.