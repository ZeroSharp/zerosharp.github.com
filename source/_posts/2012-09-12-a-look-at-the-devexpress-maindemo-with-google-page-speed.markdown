---
layout: post
title: "A look at the DevExpress MainDemo with Google Page Speed"
date: 2012-09-12 17:22
comments: true
categories: [c#, devexpress, xaf, performance]
description: An analysis of the page speed of the DevExpress MainDemo.
---
In this post I'll demonstrate how to use the [Google Page Speed](https://developers.google.com/speed/pagespeed/) tools to analyse the performance of the DevExpress XAF MainDemo.

The easiest way to run Google Page Speed is as a Chrome or Firefox plugin. Both are available [here](https://developers.google.com/speed/pagespeed/insights_extensions). I use Chrome.

Now open the DevExpress MainDemo from Visual Studio. By default it is installed to:

    C:\Users\Public\Documents\DXperience 12.1 Demos\eXpressApp Framework\MainDemo\CS\MainDemo.sln

Set the MainDemo.Web as the startup project and change the connection string in web.config if necessary. Launch the application with Chrome and login as 'Sam' (password is blank). Then press `F12` to bring up the developer tools. The last tab is the Page Speed Analysis and your browser should look like this:

{% img /images/blog/google-page-speed-001.png %}

Now click `Start Analysis` button. After a few seconds you should get something like the following results.

#### Results against the debug webserver ####

{% img /images/blog/google-page-speed-002.png %}

It's as easy as that. We now have a list of suggested improvements. The same report can be generated for any page you visit with your browser.

I get an overall score of 72 out of 100. First thing to note is that the only 'high priority' recommendation is to 'enable keep-alive' which I suspect will not be necessary when running in IIS instead of the debug webserver. 

#### Switch to use IIS ####

Actually to get the main demo to run in IIS is not altogether simple because of the security permissions required for logging in and creating or updating the schema. If you get an error message after login: 

    Login failed for user 'IIS APPPOOL\DefaultAppPool

you will need to add the IIS application pool identity to the SQL Server security. 

  * Launch SQL Server management studio and connect to the database. 
  * In the `Security\Logins` right click and select `New Login...`. 
  * Type in `IIS APPPOOL\DefaultAppPool` (you won't find it by searching) or `IIS APPPOOL\ASP.NET v4.0` depending on the security context of the application pool you are using. 
  * Select `Server Roles` and check `public` and `sysadmin` to allow the MainDemo to create the database. 

(All of this is assuming you are using a non-public instance of SQL Server for development.)

#### Results against IIS ####

The results are much better: an overall score of 93.

{% img /images/blog/google-page-speed-003.png %}

#### Other points of interest ####

Let's experiment by turning off compression in the webconfig.

    <compression 
      enableHtmlCompression="false" 
      enableCallbackCompression="false" 
      enableResourceCompression="false" 
      enableResourceMerging="false" />

The overall score drops to 62.

{% img right /images/blog/google-page-speed-004.png %}

You can alternatively use IIS's dynamic compression by setting `enableResourceMerging="true"` and the others false and adding a `urlCompression` setting as follows.

    <system.webServer>
      <urlCompression doDynamicCompression="true" />
      ...
    </system.webServer>	

(Note that you may need to install the dynamic compression module via **Control Panel/Programs/Turn Windows Features On or Off**.)

Then the analysis is back up to 93. The advantage of IIS dynamic compression in IIS 7 is that it turns itself off automatically when the CPU load is high. See [Matt Perdeck's series of articles about IIS Compression](http://www.codeproject.com/Articles/242133/Making-the-most-out-of-IIS-compression-Part-1-conf) for more information.

One mysterious point: if you navigate to the [online version of the MainDemo](http://demos.devexpress.com/XAF/MainDemo/default.aspx) and run the analysis there you will notice that compression resource merging must be turned off for some reason and the overall score is only 75. Perhaps someone from DevExpress can explain...







