---
layout: post
title: "Load Testing XAF: Part 2 - Selenium"
date: 2013-03-28 16:56
comments: true
categories: [c#, devexpress, xaf, aws, neustar]
description: How to create a Selenium test for the XAF MainDemo for use in a load test.
---
# Writing a Selenium User Test against MainDemo #

This is another post in a series about load testing XAF applications.  Previously in the series: 

* [Load Testing XAF: Overview](/load-testing-xaf-overview/)
* [Part 1: Deploying the target webserver](/load-testing-xaf-part-1-deploying/)

## Why not use DevExpress EasyTests? ##

The DevExpress recommended method of writing functional tests is to use the EasyTest functionality of the expressAppFramework. This has several advantages over other functional testing approaches.

 * It uses a domain specific language tailored for XAF making it easy to test views and actions
 * It makes it easy to interact with the DevExpress controls that are used within XAF
 * A single EasyTest can be run against both the ASP.NET and WinForms applications
 * EasyTests work against both the debug webserver and IIS

However, one feature which is not (yet) available is the ability to use EasyTests for load testing.

## Modifications to the MainDemo ##

The sample script I have written assumes the MainDemo is running with _Horizontal Navigation_ rather than vertical. You can modify the script to add support for vertical navigation or you can change Global.asax.cs Application_Start as follows:

{% codeblock lang:diff %}
protected void Application_Start(object sender, EventArgs e) 
{
    RenderHelper.RenderMode = DevExpress.Web.ASPxClasses.ControlRenderMode.Lightweight;
    ASPxWebControl.CallbackError += new EventHandler(Application_Error);
        
+    // Add the following line to default to horizontal layout
+    WebWindowTemplateHttpHandler.PreferredApplicationWindowTemplateType = DevExpress.ExpressApp.Web.Templates.TemplateType.Horizontal;
}
{% endcodeblock %}

## The Selenium script ##

Selenium is a powerful tool for automating browsers. It supports all of the major browsers and a Selenium test can be written in many different programming languages (C#, Java, Javascript, HTML, _etc._) The load testing tool (which we will come to in part 3 of this series) uses Selenium scripts written in Javascript.

We will now create and verify a simple Selenium test. The test will open the browser, login to the MainDemo and cycle through all of the tabs before logging out. The script is extremely basic. For a more realistic load test, you want a combination of scripts running, some entering data, some triggering reports, etc.

Create a \scripts subdirectory and populate it with the following code:

{% codeblock MainDemo_CycleThroughTabs.js lang:js %}

/* global test */

// Settings for Neustar:
// replace the following with the public address of the application server,
var targetHost = "http://zerosharp-maindemo.elasticbeanstalk.com/";
var virtualShare = "MainDemo.Web_deploy";

// Settings for debug webserver:
// (local script validator doesn't always work against localhost,
// so we use the excellent localtest.me instead.)
//var targetHost = "http://localtest.me:58404";
//var virtualShare = "";

// Settings for the build server or IIS:
//var targetHost = "http://localtest.me/";
//var virtualShare = "MainDemo.Web";

// Test parameters
var thinkTimeInSeconds = 3;
var timeout = 60000;
var step = 0;

// You an optionally set the simulated bandwidth for the script
// (max of 100KB/sec). A value of -1 means do not limit.
// E.g., 
// var bandwidthLimit = 50 * 1024 * 8; // 50KB/sec
var bandwidthLimit = -1;

var driver = test.openBrowser();
var selenium = driver.getSelenium();

// Support functions
function think() {
    if (thinkTimeInSeconds > 0) {
        if (!test.isValidation()) {
            test.pause(thinkTimeInSeconds * 1000);
        }
    }
}

function waitForCallbacks() {
    selenium.waitForCondition("(typeof selenium.browserbot.getUserWindow().xafHasPendingCallbacks === 'function') && (selenium.browserbot.getUserWindow().xafHasPendingCallbacks() === false);", timeout);
}


function stepLogin(username) {
    step = step + 1;
    test.beginStep("Step " + step.toString() + " - Login");
    selenium.open(targetHost + virtualShare + "/Default.aspx");
    think();
    selenium.type("xpath=//input[contains(@id,'_xaf_dviUserName_Edit_I')]", username);
    selenium.type("xpath=//input[contains(@id,'_xaf_dviPassword_Edit_I')]", "");
    selenium.click("Logon_PopupActions_Menu_DXI0_T");
    selenium.waitForPageToLoad(timeout);
    waitForCallbacks();
    selenium.assertElementPresent("Horizontal_VCC_VSL");
    selenium.waitForText("Horizontal_VCC_VSL", "Contact");
    test.endStep();
    think();
}

function stepLogoff() {
    var expectedSubstring;
    step = step + 1;
    test.beginStep("Step " + step.toString() + " - Logoff");
    selenium.click("//li[@class='dxm-item']/div[@class='dxm-content dxm-hasText']//a[@class='dx dxalink' and text()='Log Off']/..");
    selenium.waitForPageToLoad(timeout);
    expectedSubstring = "Logout.html";
    test.endStep();
}

function stepNavigateToTab(maintabCaption, tabCaption, viewCaption) {
    // viewCaption is optional
    viewCaption = (typeof viewCaption === "undefined") ? tabCaption : viewCaption;
    step = step + 1;
    test.beginStep("Step " + step.toString() + " - " + tabCaption);
    selenium.waitForElementPresent("//td[@class='dxtc' and text()='" + maintabCaption + "']");
    if (selenium.isVisible("//td[@class='dxtc' and text()='" + maintabCaption + "']")) {
        selenium.click("//td[@class='dxtc' and text()='" + maintabCaption + "']");
    }
    selenium.waitForElementPresent("//div[@class='dxm-content dxm-hasText' and starts-with(@id, 'Horizontal_NTAC_PC_M')]//a[@class='dx dxalink' and contains(text(), '" + tabCaption + "')]/..");
    selenium.click("//div[@class='dxm-content dxm-hasText' and starts-with(@id, 'Horizontal_NTAC_PC_M')]//a[@class='dx dxalink' and contains(text(), '" + tabCaption + "')]/..");
    waitForCallbacks();
    selenium.assertElementPresent("Horizontal_VCC_VSL");
    selenium.assertText("Horizontal_VCC_VSL", viewCaption);
    test.endStep();
    think();
}

function initializetest() {
    selenium.setTimeout(timeout);
    if (bandwidthLimit > 0) {
        test.setSimulatedBps(bandwidthLimit);
    }
}

(function main() {
    initializetest();

    test.beginTransaction();

    stepLogin("Sam");
    //stepNavigateToTab("Default", "Contact");
    stepNavigateToTab("Default", "Task");
    stepNavigateToTab("Default", "Department");
    stepNavigateToTab("Default", "Scheduler Event");
    stepNavigateToTab("Default", "My Details", "User - Sam");
    stepNavigateToTab("Default", "Note");
    stepNavigateToTab("Default", "Payment");
    stepNavigateToTab("Default", "Position");
    stepNavigateToTab("Default", "Resume");
    stepNavigateToTab("Default", "Role");
    stepNavigateToTab("Default", "User");
    stepNavigateToTab("Reports", "Analysis");
    stepNavigateToTab("Reports", "Reports");
    stepLogoff();
    test.closeBrowser();

    test.endTransaction();
}());

{% endcodeblock %}

## Neustar ##

In a future post we will create multiple test runners in the Amazon cloud using the [Neustar web performance tool](https://home.wpm.neustar.biz/) (formerly BrowserMob). Neustar will gather statistics about each scripts reponse times and provide a load test report including details of any test failures.

For now we will verify locally that the Selenium script above works as expected.

### Installing the Neustar local script validator ###

In order to verify that our script is supported by the Neustar framework, we need to install their [local script validator](http://static.wpm.neustar.biz/tools/local-validator.tar.gz). Download it and unzip it to a subdirectory of the MainDemo.

There are [instructions for setting up local script validation here](http://community.webmetrics.com/community/wpm/blog/2012/10/02/neustar-script-local-validator-user-guide-for-windows).

To run the script locally call the following:

    > script-validator-4.8.81\bin\validator.bat CycleThroughTabs.js -keepbrowseronerror

I had some problems getting the NeuStar script validator to work in 64-bit Windows 8. The script validator instructions recommend FireFox 12 but I am using version 19. For the record I am using:

 * DevExpress MainDemo 12.2.7
 * NeuStar localscriptvalidator 4.8.81
 * Mozilla FireFox 19
 * Java 7.0.90

You need to modify your `C:\Users\<Username>\.wpm\config.properties` file as follows:

{% codeblock config.properties %}
FF=C:\\Program Files (x86)\\Mozilla Firefox\\firefox.exe
{% endcodeblock %}

Also, for some reason, I could not get the local script validator to run against localhost. I kept getting the error:

```
WARN 03/28 12:38:28 b.n.w.a.s.JavaScrip~ - Got script exception
org.mozilla.javascript.WrappedException: Wrapped biz.neustar.webmetrics.agent.ap
i.HttpErrorException: No valid HTTP Response received while navigating to URL 'h
ttp://localhost:58404/Default.aspx' (CycleThroughTabs.js#50)
```

The easiest solution was to change the localhost address in the javascript file to the excellent localhost alternative [localtest.me](http://readme.localtest.me/).

Now when I run the script using the local validator with

    > validator cyclethroughtabs.js

I see Firefox startup after a few seconds and the script correctly cycles through all of the tabs and then exits.

We will use this scenario as the basis of a load test in the next post.
