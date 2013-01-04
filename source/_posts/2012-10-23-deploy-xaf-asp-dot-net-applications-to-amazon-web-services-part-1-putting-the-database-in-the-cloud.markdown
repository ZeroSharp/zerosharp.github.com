---
layout: post
title: "Deploy XAF ASP.NET Applications to Amazon Web Services: Part 1"
date: 2012-10-23 12:52
comments: true
categories: [c#, devexpress, xaf, aws]
description: A tutorial for publishing the DevExpress expressAppFramework MainDemo to Amazon Web Services. This part covers setting up an Amazon RDS instance of SQL Server Express and connecting to it from a local MainDemo.
---

# Part 1: Putting the Database in the Cloud #

This is the first part of a tutorial for installing the DevExpress MainDemo.Web to Amazon Web Services, but the same principles apply to any ASP.NET web application. 

This part covers creating an Amazon RDS instance running SQL Server Express and connecting a (locally running) MainDemo to it. 

At the time of writing the DevExpress version is 12.1.7.

## Amazon Web Services ##

If you have not already done so, you will need to sign up with [Amazon Web Services](http://aws.amazon.com/). There are costs associated with AWS, but the tutorial only uses very small cheap instances which cost as little as 2 cents per hour to run. Also, new customers get a load of hours for free as part of the [AWS Free Usage Tier](http://aws.amazon.com/free/). See the [AWS pricing](http://aws.amazon.com/ec2/pricing/) for more information. Don't forget to terminate your instances when you have finished.

## AWS Toolkit ##

Amazon have made it all easy by providing a Visual Studio add-in. 

Install the [AWS Toolkit for Microsoft Visual Studio](http://aws.amazon.com/visualstudio/). If you haven't used it before, then when you start a wizard it will ask you for your AWS fcredentials.

{% img /images/blog/aws/publish-to-aws-1.jpg 'figure 1' %}

The **Display Name** is the a name you give to the AWS account you are using. I set mine to _zerosharp_. This is helpful when you have multiple AWS accounts. The **Access Key** and the **Secret Key** are both available on the [security credentials page](https://aws.amazon.com/security-credentials) - you will have to click **Show** in order to display the secret key.
You can leave the account number blank.

## The database ##

Now let's provision a new SQL Server Express database.

In Visual Studio, open the AWS Explorer (`Ctrl+K,A` or in the _View_ menu). You probably want to select a region near your physical location. I chose _EU West (Ireland)_. Right-click on 'Amazon RDS' and select 'Launch Instance'.

{% img /images/blog/aws/aws-explorer-1.jpg "figure 2" %}

At the following screen, select **SQL Server Express**. Note that we could just as easily provision the Standard or Enterprise editions as well as a host of other database options.

{% img /images/blog/aws/launch-aws-rds-1.jpg "figure 3" %}

Configure the settings for the connection. Choose an instance class of **Micro** which is fine for our needs. Note the _Master User Name_ and the _Master User Password_ will be needed when we modify the MainDemo's connection string.

{% img /images/blog/aws/launch-aws-rds-2.jpg "figure 4" %}

In this screen we configure settings for the port and security. In order to connect from our local MainDemo, you need to add the permission for the your CIDR route which will let you connect to the database from your local machine. It's easy to add it to the default group later (via DB Security Groups).

{% img /images/blog/aws/launch-aws-rds-3.jpg "figure 5" %}

Set the backup options to 'No backups'. Nice to have the option though.

{% img /images/blog/aws/launch-aws-rds-4.jpg "figure 6" %}

Review the options and click 'Launch'.

{% img /images/blog/aws/launch-aws-rds-5.jpg "figure 7" %}

The database instance will be available after a few minutes. You have to be patient here - there is not much feedback, just the yellow _creating_ status. You can press 'Refresh' once in a while, but it's likely to take up to 15 minutes. Eventually you should see a green _created_ status.

## Modify the MainDemo connection string ##

Open the MainDemo in Visual Studio.  Open the web.config file and set the connection string to point to your Amazon RDS instance.

The 'User ID' and 'Password' should be the same as the ones you entered above. The address is available from the Visual Studio Properties window as 'Endpoint' when you select the DB Instance in the AWS Explorer.

{% img /images/blog/aws/aws-properties-1.jpg "figure 8" %}

My connection string looks like this:

    <add name="ConnectionString" connectionString="User ID=zerosharp;Password=password;Pooling=false;Data Source=maindemo.c5uchpz3rigs.eu-west-1.rds.amazonaws.com;Initial Catalog=MainDemo_v12.1"/>    

## Run the MainDemo locally ##

Now run the MainDemo locally. When you get to the login page, login as Sam (no password).  At this point, the MainDemo will connect to the Amazon RDS instance using the connection string we specified above and create the database (which takes at least 30 seconds on my machine). Afterwards the MainDemo will function as normal.

## Next up ##

In the [next post](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-2-publishing-maindemo/) I explain how to publish the MainDemo application itself to an EC2 instance in the Amazon cloud.

* [Part 2 - Deploying to ElasticBeanstalk](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-2-publishing-maindemo/)
* [Part 3 - Troubleshooting](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-3-troubleshooting-via-remote-desktop/)
* [Part 4 - Incremental Deployment](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-4-incremental-deployment/)
* [Part 5 - Load balancing](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-5-load-balancing/)

