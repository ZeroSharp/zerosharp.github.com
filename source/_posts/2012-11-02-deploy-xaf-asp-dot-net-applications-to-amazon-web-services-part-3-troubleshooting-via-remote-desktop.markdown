---
layout: post
title: "Deploy XAF ASP.NET Applications to Amazon Web Services: Part 3"
date: 2012-11-02 17:54
comments: true
categories: [c#, devexpress, xaf, aws]
description: A tutorial for publishing the DevExpress expressAppFramework MainDemo to Amazon Web Services. This part covers troubleshooting.
---
# Part 3: Troubleshooting #

This is the third post in a series about deploying ASP.NET applications to the Amazon cloud. 

In [Part 1](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-1-putting-the-database-in-the-cloud/) we created an Amazon RDS instance of SQL Server to act as the database for the deployment. 

In [Part 2](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-2-publishing-maindemo/) we published the DevExpress MainDemo.Web to the Amazon Elastic Beanstalk.

In this part we will look at troubleshooting methods and in particular how to connect to an EC2 instance in order to troubleshoot installations.

## The Beanstalk Events tab ##

When there is a problem with the deployment, the first place to look is in the Elastic Beanstalk events tab. You will find warnings and error messages which frequently point to the source of any problem.

{% img /images/blog/aws/aws-events-1.jpg 'figure 18' %}

Sometimes, the events window point to a log file which is automatically generated in an S3 bucket. By opening S3 in the AWS Explorer, you can find and download the log file which usually has excellent error information in it, for instance:

{% codeblock An example log file %}

    2012-11-02T12:37:13.000Z Warning 3:Web Event ASP.NET 4.0.30319.0 - Event code: 3008
    Event message: A configuration error has occurred.
    [...]

    Application information:
    Application domain: /LM/W3SVC/1/ROOT/MainDemo.Web_deploy-2-129963334311133960
    Trust level: Full
    Application Virtual Path: /MainDemo.Web_deploy
    Application Path: C:\inetpub\wwwroot\MainDemo.Web_deploy\
    Machine name: AMAZONA-C6H7CEI

    Process information:
    Process ID: 2436
    Process name: w3wp.exe
    Account name: IIS APPPOOL\DefaultAppPool

    Exception information:
    Exception type: ConfigurationErrorsException
    Exception message: Could not load file or assembly 'DevExpress.ExpressApp.AuditTrail.v12.1, Version=12.1.7.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a' or one of its dependencies. The system cannot find the file specified. (C:\inetpub\wwwroot\MainDemo.Web_deploy\web.config line 94)
      at System.Web.Configuration.CompilationSection.LoadAssemblyHelper(String assemblyName, Boolean starDirective)
      at System.Web.Configuration.AssemblyInfo.get_AssemblyInternal()
      at System.Web.Compilation.BuildManager.GetReferencedAssemblies(CompilationSection compConfig)
      at System.Web.Compilation.BuildManager.CallPreStartInitMethods(String preStartInitListPath)
      at System.Web.Compilation.BuildManager.ExecutePreAppStart()
      at System.Web.Hosting.HostingEnvironment.Initialize(ApplicationManager appManager, IApplicationHost appHost, IConfigMapPathFactory configMapPathFactory, HostingEnvironmentParameters hostingParameters, PolicyLevel policyLevel, Exception appDomainCreationException)
    [...]
    
{% endcodeblock %}

## Connecting via Remote Desktop ##

If you're still in the dark, you can use Windows Remote Desktop to connect to the EC2 machine to troubleshoot.

In the AWS Explorer, go to **Amazon EC2/Instances**. 

There are two ways of connecting. You can **Get Windows Passwords**, but this takes about an hour for the password to be decrypted. 

{% img /images/blog/aws/aws-troubleshooting-1.jpg 'figure 19' %}

Alternatively, you can **Open Remote Desktop** and then **Use EC2 keypair to logon**. You should have the _.pem_ private key file associated with the EC2 key pair that was used to launch the instance (it was created when you created your AWS account or when you launched your first EC2 instance). Whenever an EC2 instance is provisioned, it is launched with an EC2 key pair (you selected a keypair to use in the wizard above). You can open the .pem file and cut and paste the key into the remote desktop window.

You will then be logged in as an Administrator on the EC2 Instance running Windows 2008 R2. You can perform whatever operations you see fit - install software, change Control Panel options, restart IIS, _etc._ The changes will last until you terminate the instance (by deleting the Elastic Beanstalk stack for example).

{% img /images/blog/aws/aws-troubleshooting-2.jpg 'figure 20' %}

Note that it is easy to create new key pairs if you can't find the .pem file, but you cannot change the key pair of a running instance. (You should terminate and launch again with the new key pair). One nice feature of the AWS Toolkit, is that if you create a new key pair within Visual Studio, you can choose to store the private key in the Toolkit.

## Next up ##

The [next post](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-4-incremental-deployment/) will cover incremental deployment options.

* [Part 4 - Incremental Deployment](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-4-incremental-deployment/)
* [Part 5 - Load balancing](/deploy-xaf-asp-dot-net-applications-to-amazon-web-services-part-5-load-balancing/)
