---
layout: post
title: "DevExpress 2015.2 review part 1"
date: 2015-12-09 14:43
comments: true
categories: [c#, devexpress, xaf]
description: A review of the new DevExpress 2015.2 release with particular focus on the XAF ASP.NET report designer.
published: false
---
Last week, DevExpress released 2015.2.3, their second major version of the year.

There are already some good blog posts about the changes:

- [Michael Bogaerts](http://www.codeproject.com/Tips/1060260/Whats-New-for-XAF)
- [Gustavo Marzioni](http://vimarx.com/blog/92/)
- [DevExpress What's New](https://www.devexpress.com/Subscriptions/New-2015.xml?product=xaf)

Rather than repeat general overviews provided in these, this two-part blog post is more of a 'deep dive'. In particular I'll be looking at two of the new features in the expressApp Framework (XAF).
Today I'll cover the new XML serialisation in the report designer. Tomorrow's post will examine the new batch editing features.

## Reports ##

{% pullquote %}
Reporting is definitely one of the areas where XAF has progressed the most in recent versions. There is now an in-browser report designer (since 14.2) and an alternative implementation of the reports (reports v2, since 13.2). 

In this release, I was a little worried that the support for Reports v1 would be deprecated, since our production system has over 100 custom reports and we have not yet looked for an easy way to migrate these to v2. {"I'm happy to report that v1 reports are still very much present."}
{% endpullquote %}

However there are also a number of new features in v2 Reports, not least of which is the ability to store the report's layout in XML. There's currently not a lot of documentation about the XML serialization, so let's dig in and see what we can discover.

### XML serialization deep dive ###
First I ran the demo and ran the _Copy Predefined Report_ on the Contacts Report.

Then I ran the report designer and added a dummy `OnBeforePrint()` script to one of the table's cells.

```c#
private void xrTableCell8_BeforePrint(object sender, System.Drawing.Printing.PrintEventArgs e) {
  var x = 23;
}
```

Then, in the MainDemo's Updater.cs file, I placed the following code.
```c#
var reportData = ObjectSpace.FindObject<ReportDataV2>(new BinaryOperator("DisplayName", "Contacts Report") & new BinaryOperator("IsPredefined", new OperandValue(false)));
if (reportData != null)
{
    var report = ReportDataProvider.ReportsStorage.LoadReport(reportData);
    report.SaveLayoutToXml(@"C:\Temp\ContactsReport.xml");
    report.SaveLayout(@"C:\Temp\ContactsReport.repx");
}
```

This creates two output files. One of them contains the familiar _.repx_ format. The other contains the newer _.xml_ serialization.

The contents of the XML file is displayed below and is 77 lines long.

{% gist f085bf10dd65ace5d229 ContactReport.xml %}

By comparison, the [ContactReport.repx](https://gist.github.com/shamp00/f085bf10dd65ace5d229#file-contactsreport-repx) is 408 lines long and much harder to read.

As you can see the XML file is *much* smaller and simpler than the _.repx_ file. At first I didn't believe it contained all the necessary information, so I started up the MainDemo WinForms application, created a blank new report and imported the layout and the layout looks correct and the preview loads with data as expected.

### Scripts ###

What about scripts? Are they serialized in the xml version? You bet. In the XML export you can see this has been serialised near the top of the file.

```xml
ScriptsSource=
   "&#xD;&#xA;private void xrTableCell8_BeforePrint(object sender, System.Drawing.Printing.PrintEventArgs e) 
   {&#xD;&#xA;  var x = 23;&#xD;&#xA;}&#xD;&#xA;"
```

### Setup ###

In order to use this new XML serialization of report layouts, you must set the ReportsV2 module to use it.

```c#
reportsModuleV2.ReportStoreMode = ReportStoreModes.XML
```

This will most likely invalidate any reports which have already been serialized to the database. There are some notes on how to fix this [here](https://www.devexpress.com/Support/Center/Question/Details/T275363).

### Default for new projects ###

What about new projects? I created a new solution and chose *DevExpress 15.2 XAF Solution Wizard* as the type of solution and added the reports module. Now when I navigate to the `WinApplication.Designer.cs` file, I find:

```c#
//
// reportsModuleV2
//
this.reportsModuleV2.ReportDataType = typeof(DevExpress.Persistent.BaseImpl.ReportDataV2);
this.reportsModuleV2.ReportStoreMode = DevExpress.ExpressApp.ReportsV2.ReportStoreModes.XML;
```

So the default storage method for new projects is now XML.

### Conclusion ###

The XML serialisation looks like a considerable upgrade to the mechanism for storing, loading and saving reports. Now I just need to find a good way of converting my existing v1 reports... Perhaps a future blog post.

## Coming up ##

Tomorrow I'll be looking at the new [batch editing in grids]() in more detail.


