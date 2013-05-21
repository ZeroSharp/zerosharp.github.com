---
layout: post
title: "Making XAF reports even better - Part 1"
date: 2013-05-21 14:38
comments: true
categories: [c#, devexpress, xaf, reports]
description: First part of a series about improving the way DevExpress XafReports are managed.
---
{% pullquote %}
The ability to create reports using a report writer is a very powerful feature of [DevExpress XAF](http://www.devexpress.com/Products/NET/Application_Framework/), but there are some limitations which are particularly cumbersome to deal with in complex project.

One of the projects I work on has over 100 reports in it. Even though we make use of unit tests to ensure they are not broken, the maintenance of the code in the embedded scripts is particularly difficult to manage.

* XafReports are _.repx_ files which are usually loaded into the report table during the database update routine. They are a subclass of XtraReports which with some added restrictions.
* Any scripts are stored as a string or serialized to a _resources_ property.
* The report writer is available only in the Windows Forms application. This must be used whenever a change is made to a report. The modified report must be exported as a repx file and then added to the module as an embedded report. The procedure is [described here](http://documentation.devexpress.com/#Xaf/CustomDocument2786).

These aspects of XAF reports give rise to several development headaches. 

* While script syntax can be checked within the report writer at design time (via the Validate button in the scripts tab), the script code is still brittle.
* Errors that result from Script syntax is sometimes only discovered at run time (you can write a unit test to check during build, but we really want to the compiler to tell us).
* Refactoring any classes used in scripts can result in considerable work with the report writer to apply any changes to the code within the scripts.
* Version control diff comparisons and merging are impossible.

{"The aim of these posts is to provide a two-way conversion process between .repx and C# files."} In order to accomplish this we'll be relying on Visual Studio's excellent T4 templating engine along with Oleg Sych's T4 Toolbox extension.
{% endpullquote %}

## Installing T4Toolbox ##

[T4 Text Templates](http://msdn.microsoft.com/en-us/library/vstudio/bb126445.aspx) is a template based code generation framework which is included with Visual Studio. On top of this [Oleg Sych](http://www.olegsych.com/) provides a Visual Studio extension called [T4 Toolbox](http://visualstudiogallery.msdn.microsoft.com/7f9bd62f-2505-4aa4-9378-ee7830371684) which adds some additional features.

Install T4 Toolbox by selecting **Tools/Extensions and Updates** from Visual Studio and searching for it.

{% img /images/blog/xaf-report-sync-001.png %}

## The ReportSync MainDemo ##

Next, download the modified MainDemo application from my [GitHub repository](https://github.com/ZeroSharp/Xaf_MainDemo_ReportSync) and open it in Visual Studio.

First lets look at the embedded reports which I have modified slightly so that they include scripts. I added these scripts via the MainDemo.Win application.

Let's look at the _ContactsGroupByPosition.repx_ file. You will find that there is a section:

{% codeblock lang:csharp %}
this.ScriptsSource = "\r\nprivate void xrLabel4_BeforePrint(object sender, System.Drawing.Printing.PrintE" +
    "ventArgs e) {\r\n\txrLabel4.Text = xrLabel4.Text + \" Test!\";\r\n}\r\n";
{% endcodeblock %}

In this case, the script has been saved as a string. The other report, which has only slightly more complex script code looks like this:

{% codeblock lang:csharp %}
private System.Resources.ResourceManager resources {
    get {
        if (_resources == null) {
            string resourceString = @"zsrvvgEAAACRAAAAbFN5c3RlbS5SZXNvdXJjZXMuUmVzb3VyY2VSZWFkZXIsIG1zY29ybGliLCBWZXJzaW9uPT.....5kIFtEdWVEYXRlXSA8PSAnQEN1cnJlbnREYXRlJwABEFRhc2tzU3RhdGVSZXBvcnQ=";
            this._resources = new DevExpress.XtraReports.Serialization.XRResourceManager(resourceString);
        }
        return this._resources;
    }
}
// ...
this.ScriptsSource = resources.GetString("$this.ScriptsSource");
{% endcodeblock %}

Here, the scripts are not even in plain text. They have been serialised to the _resources_ property.

## The MainDemo.Reports assembly ##

You will find a new assembly **MainDemo.Reports** which contains a T4 template _RepxToCSharp.tt_. This is a T4 template which will search for repx files and transform them into much more helpful plain C#.

The template will run every time it is saved. Currently, it depends on code within the MainDemo.Reports assembly, so make sure you have compiled it in Debug mode. Then open the _RepxToCSharp.tt_ and press `Ctrl+S` to save (and run the T4 transformation).

## The output ##

The template will generate two types of output. First, it generates the following report which you should find in _RepxToCSharp.txt_

{% codeblock lang:text %}
(This is an automatically generated file which should be excluded from version control)

Summary of process repx files
=============================
Total repx files found                                      :  2
  Total reports generated                                   :  2
  Total reports skipped because unchanged                   :  0

Time elapsed: 00:00:02.1762029
{% endcodeblock %}

In addition, each repx will have been transformed into two correpsonding files. All the generated files are highlighted in yellow:

{% img /images/blog/xaf-report-sync-002.png %}

Now the scripts have been deserialized from the repx and put in a partial class and the remainder of the repx has been transformed into a corresponding `XafReport` descendant. See for instance, _ContactsGroupedByPosition.cs_ (which stored its scripts as a string) is as follows:

{% codeblock lang:csharp ContactsGroupedByPosition.cs %}
public partial class _ContactsGroupedByPosition
{
    // -- Start of embedded scripts -- 
    
    private void xrLabel4_BeforePrint(object sender, System.Drawing.Printing.PrintEventArgs e) {
    	xrLabel4.Text = xrLabel4.Text + " Test!";
    }
    
    // -- End of embedded scripts --    
}
{% endcodeblock %}

And _TasksStateReport.cs_ is now like this

{% codeblock lang:csharp TasksStateReport.cs %}
public partial class _TasksStateReport
{ 
    // -- Start of embedded scripts -- 
   
    private void xrLabel1_BeforePrint(object sender, System.Drawing.Printing.PrintEventArgs e) {
        // This is a test
        xrLabel1.Text = "Hello";
    }
    
    private void xrLabel2_BeforePrint(object sender, System.Drawing.Printing.PrintEventArgs e) {
        xrLabel2.Text = GetLabel2Text();
    }
    
    public string GetLabel2Text()
    {
    	return "Label 2!";
    }
    
    // -- End of embedded scripts --    
}
{% endcodeblock %}

## A note about performance ##

The process of transforming the repx into C# is quite quick (a couple of seconds per _repx_), but when you have dozens of reports, it can quickly be tiresome. Therefore, there is a performance optimisation which checksums the repx and skips the transformation if it has not changed.

(In a future version, we will also use a similar checksum in the other direction to determine whether the scripts have been modified).

## Already much better ##

Now we have much more useful source files. Versions can be compared easily. The compiler will immediately inform us of any problems with the scripts within our reports.	

This is work in progress. Next up, I will be adding the 'reverse'. That is, a new transformation template which looks for scripts which have changed and 'injects' them back into the original _repx_ file.

