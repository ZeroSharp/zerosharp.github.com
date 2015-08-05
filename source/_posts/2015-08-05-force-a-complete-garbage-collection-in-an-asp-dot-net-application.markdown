---
layout: post
title: "Force a complete garbage collection in an ASP.NET application."
date: 2015-08-05 09:26
comments: true
categories: [c#, devexpress, asp.net, xaf]
description: How to force a full garbage collection within an XAF ASP.NET application.
---
How can I force a full garbage collection easily within an ASP.NET application? The method here is for XAF web applications but the same approach should work with any ASP.NET app. 

{% pullquote %}
{"First up: Never mess with the .NET garbage collector."} 

I sometimes mess with the garbage collector in .NET when I'm trying to pin down some memory problem. Also, after [a load test](/load-testing-xaf-part-1-deploying/), I prefer to force the garbage collector to collect everything it can so that I can check that the memory drops as expected. 
{% endpullquote %}

Garbage collection in .NET is complex and it is hard to be sure you've done it correctly. [This ancient article by Tess Ferrandez](/// See http://blogs.msdn.com/b/tess/archive/2006/02/02/net-memory-my-object-is-not-rooted-why-wasn-t-it-garbage-collected.aspx
) pointed me in the following direction.

```c#
void ForceGarbageCollection()
{
    /// This will garbage collect all generations (including large object), 
    GC.Collect(3);
    /// then execute any finalizers
    GC.WaitForPendingFinalizers();
    /// and then garbage collect again to take care of all the objects that had finalizers.            
    GC.Collect(3);
}
```

How can we easily trigger this routine in an XAF web application? First, add the following to the _Global.asax.cs_ file:

```c# Global.asax.cs
protected void Application_BeginRequest(object sender, EventArgs e) {
    var shouldForceGarbageCollection = Request.QueryString["ForceGC"] != null;
    if (shouldForceGarbageCollection)
    {
        ForceGarbageCollection();
        Response.Write("<H1>Forced garbage collection OK!</H1>");
    }
}
```

Then whenever you want to trigger a garbage collection, visit the following URL:

    http://.../Login.aspx?ForceGC=1

You should see the text **'Forced garbage collection OK'** in the top left of the browser window.