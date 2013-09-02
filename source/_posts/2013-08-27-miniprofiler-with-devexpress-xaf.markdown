---
layout: post
title: "MiniProfiler with DevExpress XAF"
date: 2013-08-27 09:14
comments: true
categories: [c#, devexpress, xaf, performance]
description: How to add MiniProfiler to the XAF MainDemo web application.
---
In this post I will demonstrate how to add [MiniProfiler](http://miniprofiler.com/) to the XAF MainDemo web application.

MiniProfiler is a simple fast profiler with a pretty user interface. It is fast because it only profiles code that you have explicitly decorated with the `MiniProfiler.Step()` method. It was designed by the team at [StackOverflow](http://stackoverflow.com/).

First, add the MiniProfiler NuGet package to the MainDemo.Web project. Then add a placeholder to _default.aspx_ just before the last `<body>` tag.

```html
    <!-- MiniProfiler -->
    <!-- Include jquery here to avoid a bug in MiniProfiler. -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
    <asp:PlaceHolder ID="mp" runat="server">
      <%= StackExchange.Profiling.MiniProfiler.RenderIncludes() %>
    </asp:PlaceHolder>
</body>
</html>
```

(MiniProfiler uses jQuery, but it does not usually require XAF to include it since it will automatically retrieve it if missing. Unfortunately there is currently [a bug](http://community.miniprofiler.com/permalinks/149/jquery-is-undefined) which causes a '_jQuery is undefined_' javascript error when initially launching the application. The easiest workaround I found is to explicitly include jQuery before calling `RenderIncludes()`. Hopefully this will be fixed in a future version of MiniProfiler.) 

In _global.asax.cs_ add the following to the `Application_Start` method.

```diff
protected void Application_Start(object sender, EventArgs e) {
    RenderHelper.RenderMode = DevExpress.Web.ASPxClasses.ControlRenderMode.Lightweight;
+   MiniProfilerHelper.RegisterPathsToIgnore();
    ASPxWebControl.CallbackError += new EventHandler(Application_Error);
    // etc...
```
and modify `BeginRequest` and `EndRequest` as follows:
```diff
protected void Application_BeginRequest(object sender, EventArgs e) {
+   if (MiniProfilerHelper.IsEnabled())
+   {
+       MiniProfiler.Start();
+   }
    string filePath = HttpContext.Current.Request.PhysicalPath;
    if(!string.IsNullOrEmpty(filePath)
        && (filePath.IndexOf("Images") >= 0) && !System.IO.File.Exists(filePath)) {
        HttpContext.Current.Response.End();
    }
}

protected void Application_EndRequest(Object sender, EventArgs e)
{
+   if (MiniProfilerHelper.IsEnabled())
+       MiniProfiler.Stop();
}        
```
Now we implement a helper class which determines whether profiling is enabled and which URLs to profile. We can use a variety of methods, the cookie probably being the most versatile one, but for the moment, the `IsEnabled()` function always returns true.
```c#    
public static class MiniProfilerHelper
{
    public static bool IsEnabled()
    {
        // While we are testing let's always return true
        return true;

        // We should not profile if we are EasyTesting
        if (TestScriptsManager.EasyTestEnabled == true)
            return false;

        // We could choose to profile only local requests
        if (HttpContext.Current.Request.IsLocal)
            return true;

        // Or based on a cookie
        HttpCookie miniProfileCookie = HttpContext.Current.Request.Cookies["MainDemoMiniProfiler"];
        return miniProfileCookie != null && miniProfileCookie.Value != "0";
    }

    // Optionally ignore some paths to prevent the output being too busy.
    public static void RegisterPathsToIgnore()
    {
        if (!NetSecuritySettings.IsProfilingAllowed())
            return;

        List<String> ignoredByMiniProfiler = new List<String>(MiniProfiler.Settings.IgnoredPaths);
        // these are a substring search so wildcards are not supported
        ignoredByMiniProfiler.Add("SessionKeepAliveReconnect.aspx");
        ignoredByMiniProfiler.Add("TemplateScripts.js");
        ignoredByMiniProfiler.Add("EasyTestJavaScripts.js");
        ignoredByMiniProfiler.Add("MoveFooter.js");
        ignoredByMiniProfiler.Add("ImageResource.axd");
        MiniProfiler.Settings.IgnoredPaths = ignoredByMiniProfiler.ToArray(); 
    }
}
```

Done. Now whenever you run the web application, you get timing statistics for the loading of the assets. They appear as little clickable 'chiclets' in the top left of the browser page.

{% img /images/blog/xaf-miniprofiler-1.png %}

However, the real strength of MiniProfiler comes with the ability to add your own profiling steps. Let's say we want to know exactly what percentage of the load takes place in the `OnLoad` event. Then we add the following to _default.aspx.cs_ in order to add a 'step' to the MiniProfiler breakdown.

```c#
protected override void OnLoad(EventArgs e)
{
    var profiler = MiniProfiler.Current; // it's ok for this to be null
    using (profiler.Step("ASP.NET: Page_Load(Default)"))
    {
         base.OnLoad(e);
    }
}
```

Now, the output is a good deal richer. Also, note that if the MiniProfiler assembly is missing from the web application's _bin_ directory, the profiling is ignored completely without error.

{% img /images/blog/xaf-miniprofiler-2.png %}

As another example, let's profile the _FindBySubject_ controller action.

Add the MiniProfiler NuGet package to the MainDemo.Module project. Then modify the FindBySubjectController.cs as follows

```diff
private void FindBySubjectAction_Execute(object sender, ParametrizedActionExecuteEventArgs e) 
{
+   var profiler = MiniProfiler.Current;
+   using (profiler.Step("FindBySubject")) // doesn't matter if profiler is null
+   {
        IObjectSpace objectSpace = Application.CreateObjectSpace();
        string paramValue = e.ParameterCurrentValue as string;
        if (!string.IsNullOrEmpty(paramValue))
        {
            paramValue = "%" + paramValue + "%";
        }
        object obj = objectSpace.FindObject(((ListView)View).ObjectTypeInfo.Type,
            new BinaryOperator("Subject", paramValue, BinaryOperatorType.Like));
        if (obj != null)
        {
            e.ShowViewParameters.CreatedView = Application.CreateDetailView(objectSpace, obj);
        }
+   }
}       
```     

{% img right /images/blog/xaf-miniprofiler-3.png %}

Now navigate to the _Tasks_ list view and enter some text where it says 'Type Subject...'. You should see a new chiclet appear which contains the timing details as shown here.

MiniProfiler is a great tool for providing helpful profiling benchmarks, even in production. It's often difficult to measure when a remote user complains to support that the site seems slow. How slow is slow? In a production environment, you can turn on MiniProfiler for the user (by setting a cookie for instance) and then ask them to share their profiling information for some basic operations. This information can be invaluable in determining where the fault lies.

You can play around with [the sample solution](https://github.com/ZeroSharp/Xaf_MainDemo_MiniProfiler) up on GitHub.
