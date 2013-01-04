---
layout: post
title: "Deploy XAF ASP.NET Applications to Amazon Web Services: Part 4"
date: 2012-11-05 14:16
comments: true
categories: [c#, devexpress, xaf, aws]
description: A tutorial for publishing the DevExpress expressAppFramework MainDemo to Amazon Web Services. This part discusses incremental deployment to the Amazon cloud.
---
## Part 4: Incremental Deployment ##

This is the fourth post in a series about deploying ASP.NET applications to the Amazon cloud. In the first three posts covered deploying the XAF ASP.NET MainDemo to the Amazon cloud ([Part 1](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-1-putting-the-database-in-the-cloud/), [Part 2](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-2-publishing-maindemo/), [Part 3](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-3-troubleshooting-via-remote-desktop/)).

When we deployed to the Elastic Beanstalk in [Part 2](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-2-publishing-maindemo/), we chose _not_ to enable incremental deployment.

{% img /images/blog/aws/launch-aws-beanstalk-2.jpg 'figure 11' %}

I want to explain this decision further.

If you choose not to deploy incrementally your deployment will take longer because the entire web deployment package needs to be uploaded every time you re-publish which takes about 4-5 minutes on my connection. The incremental deployment option creates a git repository in the target environment so that only the modified files are re-deployed. If you are frequently making changes and redploying, you can save considerable time. A re-deployment takes only a few seconds.

However, if you try to deploy from the default MainDemo location with incremental deployment enabled, you will probably get the following error

    Commencing deployment for project MainDemo.Web
    ...building deployment package obj\Release\Package\Archive...
    ...deployment package created at C:\Users\Public\Documents\DXperience 12.1 Demos\eXpressApp Framework\MainDemo\CS\MainDemo.Web\obj\Release\Package\Archive
    ...build of project archive completed succesfully
    ...starting deployment to AWS Elastic Beanstalk environment 'zerosharp-maindemo'
    ...starting incremental deployment to environment 'zerosharp-maindemo'
    Deployment to AWS Elastic Beanstalk failed with exception: Exception caught during execution of add command,
    Inner Exception Message: The specified path, file name, or both are too long. The fully qualified file name must be less than 260 characters, and the directory name must be less than 248 characters.
    ...
    Deployment to AWS Elastic Beanstalk environment 'zerosharp-maindemo' did not complete successfully

The problem is with the NGit libraries that the AWS Toolkit uses. Presumably a fix will emerge, but the simplest workaround is to move the application to a shorter path. Then you can enable incremental deployment when you publish:

{% img /images/blog/aws/launch-aws-beanstalk-incremental-2.jpg 'figure 21' %}

Note that you can toggle incremental deployment whenever you publish, so you're not locked in either way.

## Next up ##

In the [next post](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-5-load-balancing/) we will enable load balancing (which is easy) and discuss the problems with autoscaling XAF applications (which is not).