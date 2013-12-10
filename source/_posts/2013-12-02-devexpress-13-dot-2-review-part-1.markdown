---
layout: post
title: "DevExpress 13.2 Review - Part 1"
date: 2013-12-02 16:23
comments: true
categories: [c#, devexpress, xaf]
description: A review of the new DevExpress 13.2 release with particular focus on XAF Reports V2.
---
The first version of XAF I ever installed was called _eXpressApp-1.0.CTP2.exe_ in July 2006! It has certainly come a long way since then. 

This post is the first part of an overview of the brand new version XAF 13.2 which will be released any day now. This part is an in-depth review of the new report writer features (still in beta).

## Reports V2 ##
Prior to this version, XAF reports were not source code. They were objects serialized to a file and then loaded into the database during initialisation. This has a lot of shortcomings some of which I have addressed in previous [blog posts](/making-xaf-reports-even-better-part-1/).

But now finally it is now possible to build XAF reports directly from Visual Studio. DevExpress is calling this [Reports V2](https://community.devexpress.com/blogs/eaf/archive/2013/11/08/easy-large-reports-in-13-2-ctp-of-reportsv2-module.aspx) and it largely supersedes my previous workarounds. The immediate advantages of Reports V2 are:

* Build time syntax checking
* Easily merge/compare report versions
* Painless version control of reports
* Better unit testing possibilities

Let's have a look at the details. DevExpress have provided us with a new sample project called _ReportsV2Demo_. The interesting part of the project solution is here:

{% img /images/blog/devexpress-13-2-review-003.png %}

Notice that there are two folders containing reports. _PredefinedEditableReports_ contains V1 reports. That is, the report is serialised as a string and loaded into the database during initialisation. If we look at the source for one of these, here is what it looks like:

```c#
namespace ReportV2Demo.Module.PredefinedEditableReports {
    public class OrdinalReportResource {
        public static string ReportLayout = @"
/// <XRTypeInfo>
///   <AssemblyFullName>DevExpress.XtraReportsVSuffix, Version=dllVersion, Culture=neutral, PublicKeyToken=dllPublicKeyToken</AssemblyFullName>
///   <AssemblyLocation>C:\Windows\Microsoft.Net\assembly\GAC_MSIL\DevExpress.XtraReportsVSuffix\v4.0_dllVersion__dllPublicKeyToken\DevExpress.XtraReportsVSuffix.dll</AssemblyLocation>
///   <TypeName>DevExpress.XtraReports.UI.XtraReport</TypeName>
///   <Localization>en-US</Localization>
///   <Version>VersionShort</Version>
///   <References>
///     <Reference Path=""C:\Windows\Microsoft.Net\assembly\GAC_MSIL\DevExpress.Persistent.BaseVSuffix\v4.0_dllVersion__dllPublicKeyToken\DevExpress.Persistent.BaseVSuffix.dll"" />
///     <Reference Path=""C:\Windows\Microsoft.Net\assembly\GAC_MSIL\System\v4.0_4.0.0.0__b77a5c561934e089\System.dll"" />

/// etc.
```
The second folder, _PredefinedReadonlyReports_ contains the V2 reports. With these reports, we can right-click and __View Designer__ and up pops the XtraReport designer.

{% img /images/blog/devexpress-13-2-review-004.png %}

Now I can make changes to my report within Visual Studio. In theory I can preview the report and view the HTML output. However, in the beta (13.2.4) the Preview view prints the report correctly but fails to fetch any data so all the fields are empty (maybe this is by design).

(Actually, I think the names of these folders could be improved. Maybe XafReports vs. XtraReports. Or RuntimeReports vs. DesigntimeReports.)

With respect to event scripts which in the past were particularly arduous to maintain, we now have two options.

### Scripts option 1 - Use serialized scripts
 Firstly, you can continue to use the scripts as before. You can use Visual Studio's _Properties_ window to navigate to the script you need and then add the necessary code in the custom editor. 

{% img /images/blog/devexpress-13-2-review-005.png %}

Unfortunately, this is not the best from a maintenance perspective. The Scripts code editor view is not at all comparable to Visual Studio. No Intellisense. No CodeRush. In addition, the script itself gets serialized to a string. 

```c#
this.ScriptsSource = "\r\nprivate void xrLabel2_BeforePrint(object sender,System.Drawing.Printing.PrintE" +
    "ventArgs e) {\r\n    string something = \"something\";\r\n}\r\n";
```
So unfortunately this approach does not improve things much with regard to compile-time syntax checking or code merging/version control.

### Scripts option 2 - Move the scripts to the C# file
Now that the report is more like an `XtraReport`, we can attach an event in code. To test it I'll replace the output of the _First name_ with the current time.

```c#
public partial class XtraReportOrdinary : DevExpress.XtraReports.UI.XtraReport
{
    public XtraReportOrdinary()
    {
        InitializeComponent();
        xrLabel2.BeforePrint += xrLabel2_BeforePrint;
    }

    private void xrLabel2_BeforePrint(object sender, System.Drawing.Printing.PrintEventArgs e)
    {
        // Today's date and time
        xrLabel2.Text = DateTime.Now.ToString("G");
    }
}
```

Now let's see if it works. We start up the web application and run the report.

{% img /images/blog/devexpress-13-2-review-006.png %}

Excellent! You can see the report has successfully run the event and is displaying the current time instead of the first name. Here are the advantages of handling events this way.

* If there's a mistake in an event, the project will not compile until it's fixed.
* The code is more readable.
* The source code of the events can easily be compared with older versions.
* The source code can be merged much more easily when handling version control conflicts.
* Unit testing of the events becomes feasible.

### Runtime tailoring ###
Can an end-user still make a copy and tailor the report as they could with the older versions? Yes they can! Let's start up the WinForms client and have a look.

{% img /images/blog/devexpress-13-2-review-007.png %}

Each of the _PredefinedReadonlyReports_ has a new option _Copy predefined report_. Also, notice that the _Show Report Designer_ is greyed out. Now compare with the copy.

{% img /images/blog/devexpress-13-2-review-008.png %}

This time it is the _Copy predefined report_ is greyed out and the familiar _Show Report Designer_ has become available. (A little suggestion to the DevExpress crew. It would be better if the new copy got renamed to _Copy of Inplace Report_ like Windows does when copying a file). Now we can open the Report Designer and customise the report further. Fantastic!

Let's check what happened to our events?

{% img /images/blog/devexpress-13-2-review-009.png %}

Well, the serialized event was copied correctly, but the script that we put in the C# file has not been copied. A bit unfortunate - that event would have to be re-written or copied manually from the C# source file, but in most cases, additional tailoring of reports is a one-off occasional activity. For me at least, the benefits of having more maintainable code outweigh the inconvenience of losing the events when cloning a report.

## Conclusions ##

Reports V2 is certainly the most significant improvement to the XAF reporting engine yet. I'm sure there are a few beta issues to iron out and a few further improvements down the line. In this post, I haven't looked at features such as views, sub-reports and custom reporting components, all of which deserve further investigation. 

In my [main development project](http://www.coprocess.com/), there are currently 110 different reports and maintaining them takes some effort. Reports V2 will help considerably. Hats off to the DevExpress team.

## Next up ##

In my next post I'll be taking a look at the new soft validation along with some other 13.2 improvements.