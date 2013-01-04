---
layout: post
title: "Deploy XAF ASP.NET Applications to Amazon Web Services: Part 2"
date: 2012-10-31 12:52
comments: true
categories: [c#, devexpress, xaf, aws]
description:  A tutorial for publishing the DevExpress expressAppFramework MainDemo to Amazon Web Services. This part covers publishing the MainDemo application to the AWS Elastic Beanstalk.
---
# Part 2: Publishing MainDemo #

This is the second post in a series about deploying ASP.NET applications to the [Amazon cloud](http://aws.amazon.com/). 

In [Part 1](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-1-putting-the-database-in-the-cloud/) we created an Amazon RDS instance of SQL Server to act as the database for the deployment. Make sure this is up and running before continuing.

This part guides you through publishing the [DevExpress XAF](http://www.devexpress.com/Products/NET/Application_Framework/) MainDemo application to the Amazon Elastic Beanstalk.

## Amazon Elastic Beanstalk ##

{% pullquote %}
{" Amazon Elastic Beanstalk provides automatic capacity provisioning, load balancing, auto-scaling, and application health monitoring. "} **That sounds like a lot of features we don't really need** for the MainDemo, so let me explain.

With Amazon Web Services, the choices are endless. An Amazon EC2 instance is a virtual machine running an actual Windows (or other) operating system that you can connect to with Remote Desktop and configure however you want. It might seem that the simplest would be to put everything on a 'single machine' instance. However, the default 'single machine' does not have SQL Server Express installed. We could connect to it and install it, but it ends up being a lot of extra steps. There are other Amazon Machine Images (AMIs) that include SQL Server Express, but these don't support automatic deployment from Visual Studio.
{% endpullquote %}

In fact (and Amazon should advertise this better), the Elastic Beanstalk is designed to make it very simple to deploy applications quickly **even if you have no intention of ever needing more than a single little instance**. It is the recommended starting point for all new deployments regardless of size. Ignore the long list of amazing additional features for now, (but one day you might need them.)

## Prepare the MainDemo for deployment ##

First we need to fix a few problems with the MainDemo. (These will cause deployment in problems any environment, not just Amazon Web Services.)

 - remove the reference to _stdole_ in the web.config _&lt;assemblies&gt;_ section
 - add a reference to the project _DevExpress.XtraPrinting.v12.1_
 - add a reference to the project _DevExpress.XtraNavBar.v12.1_
 - add a reference to the project _DevExpress.Utils.v12.1.UI_
 - set the _CopyLocal_ flag to true for all DevExpress assemblies

(Some of these might have been fixed if you are using a newer release of XAF than 12.1.7.)

Also, if there are any errors, we don't get much useful feedback unless we turn custom errors off. Go to the web.config and add a tag just before the end of the _&lt;system.web&gt;_ section.

        ...
          <customErrors mode="Off"/>
        </system.web>

Make sure you can build without errors.

## Deploy to Elastic Beanstalk ##

You might like to switch to **Release** mode since we will be publishing. Right click on the MainDemo.Web project in the Solution Explorer and select **Publish to AWS...**

{% img /images/blog/aws/aws-solution-explorer-1.jpg 'figure 9' %}

This opens the AWS publishing wizard. Select the **AWS Elastic Beanstalk** template.  I have also selected to deploy in the **EU West (Ireland)** region. Click **Next**.

{% img /images/blog/aws/launch-aws-beanstalk-1.jpg 'figure 10' %}

Make sure you uncheck the **Deploy application incrementally** checkbox (see the upcoming Part 4 for more information on this option).

{% img /images/blog/aws/launch-aws-beanstalk-2.jpg 'figure 11' %}

The next few screens are self-explanatory.

{% img /images/blog/aws/launch-aws-beanstalk-3.jpg 'figure 12' %}

(Note the keypair property is referred to in [Part 3](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-3-troubleshooting-via-remote-desktop/) in the section about connecting via Remote Desktop.)

{% img /images/blog/aws/launch-aws-beanstalk-4.jpg 'figure 13' %}

{% img /images/blog/aws/launch-aws-beanstalk-6.jpg 'figure 14' %}

The following screen appears only if we have active RDS instances running (whcih you should if you followed [Part 1](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-1-putting-the-database-in-the-cloud/)). It allows us to add a permission for the application to access the database. This can also be done manually via the *Security Groups* section in the AWS Explorer.

{% img /images/blog/aws/launch-aws-beanstalk-7.jpg 'figure 15' %}

The last screen summarizes all the choices so far.

{% img /images/blog/aws/launch-aws-beanstalk-8.jpg 'figure 16' %}

After you click **Launch**, you can follow the progress in the Amazon Web Services output window. It will upload the MainDemo web deployment package to a temporary S3 bucket. It will then provision an EC2 instance and install the application to it. It will also provision an Elastic Load Balancer and connect up all the pieces. This will all take a few minutes.

## Navigate to the Deployed MainDemo ##

Launch your browser and navigate to the instance web address which will look something like **http://zerosharp-maindemo.elasticbeanstalk.com/**. It should forward you to the login page. Login as Sam. Depending on whether the database needs to be created, this may take a couple of minutes and may even time out. I had to wait a minute and then click Login again before I was in.

When you have finished **don't forget to terminate both the Elastic Beanstalk application and the DB Instance** (right click and 'Delete' from the AWS Explorer) otherwise you will be charged for the time the machines are provisioned. You can always check that everything has terminated properly in the [AWS Console](https://console.aws.amazon.com/).

## Next up ##

In the next posts we will look some additional details:

* [Part 3 - Troubleshooting](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-3-troubleshooting-via-remote-desktop/)
* [Part 4 - Incremental Deployment](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-4-incremental-deployment/)
* [Part 5 - Load balancing](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-5-load-balancing/)

#### References ####

* The [Amazon tutorial for the AWS Visual Studio Toolkit](https://www.youtube.com/watch?v=z-N0z5K_WFI)
* Nathanial Woolls has written an excellent similar article on hosting [XAF ASP.NET applications on Azure](http://nwoolls.wordpress.com/2012/09/20/hosting-xaf-asp-net-projects-using-azure-web-sites/).
