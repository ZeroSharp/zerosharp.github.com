---
layout: post
title: "A web UI performance tip for XAF web applications"
date: 2014-03-31 10:16
comments: true
categories: [c#, devexpress, xaf]
description: A description of the DelayedViewItemsInitialization toggle in the DevExpress expressAppFramework.
---
The purpose of this post is to raise your awareness of a toggle which exists in the [DevExpress XAF framework](http://www.devexpress.com/xaf/) which can significantly improve UI performance in the web application.

{% pullquote %}
The biggest XAF project I work with has one very complex business object. The layout for this screen includes about 100 properties, several nested tabs, some custom editors, several collection properties and a whole lot of [Conditional Appearance](https://documentation.devexpress.com/#Xaf/CustomDocument3286) rules. It was very sluggish to navigate - it was taking several seconds to load the detail view and then it was very slow switching between tabs. Switching to edit mode was also slow.

Last week, I almost accidentally changed the value of `DelayedViewItemsInitialization` to `false` and noticed that the UI speed was much much better. {"In fact the general responsiveness of the entire web-application seems much better."}

In order to give it a whirl, navigate to the WebApplication.cs file (normally in the ApplicationCode subfolder of your web project) and modify the constructor as follows:

{% endpullquote %}

```c#
public MainDemoWebApplication() {
    InitializeComponent();
    this.DelayedViewItemsInitialization = false;
}
```

Certainly this is not without consequences, and I would urge a careful reading of the [relevant documentation](https://documentation.devexpress.com/#xaf/DevExpressExpressAppXafApplication_DelayedViewItemsInitializationtopic). To be honest, I still don't really understand why my detail view is so much slower without this change. I have tried to isolate the cause without much success and I will update this post if I find anything new. But if some of your detail views seem overly slow, certainly try it out.