---
layout: post
title: "An XAF workaround for 'Prevent this page from creating additional dialogs.'"
date: 2016-05-16 13:51
comments: true
categories: [c#, devexpress, xaf, chrome]
description: A workaround to detect and fix when a user has checked the 'prevent this page from creating additional dialogs' checkbox in Chrome or Firefox.
---
Both Chrome and Firefox have a 'feature' which allows a user to ignore future calls to `confirm()`. Once this has been checked, any subsequent calls to `confirm()` return false immediately without showing the window. 

In XAF, this behaviour prevents the application from working correctly. For instance, it becomes impossible to confirm a deletion.

{% img /images/blog/prevent-this-page-001.png %}

The following controller provides a work around. It injects some javascript into the page wrapping the call to `confirm()`. The new code detects when the __Prevent this page from creating additional dialogs__ checkbox has been checked and returns true instead. The confirmation window still does not appear, but the XAF application works as if the user had pressed confirm instead of cancel.

{% codeblock lang:c# HandleDisabledConfirmationsController.cs %}
using DevExpress.ExpressApp;
using DevExpress.ExpressApp.Web;
using DevExpress.Web.Internal;

namespace NetModule.Web.Controllers
{
    public class HandleDisabledConfirmationsController : Controller
    {
        public static bool IsChrome
        {
            get
            {
                return RenderUtils.Browser.IsChrome;
            }
        }

        public static bool IsFirefox
        {
            get
            {
                return RenderUtils.Browser.IsFirefox;
            }
        }

        protected override void OnFrameAssigned()
        {
            base.OnFrameAssigned();
            if (IsChrome || IsFirefox)
            {
                WebWindow window = Frame as WebWindow;
                if (window != null)
                {
                    window.CustomRegisterTemplateDependentScripts += window_CustomRegisterTemplateDependentScripts;
                }
            }
        }

        private void window_CustomRegisterTemplateDependentScripts(object sender, CustomRegisterTemplateDependentScriptsEventArgs e)
        {
            // wrapper for 'confirm'
            WebWindow window = (WebWindow)sender;

            // Detect the user has checked the 'prevent this page from creating additional dialogs'.
            // In which case assume all confirmations are accepted, rather than the default rejected
            var confirmWrapper = @"
<script>
window.nativeConfirm = window.confirm;
window.confirm = function(message)
{
    var timeBefore = new Date();
    var confirmBool = nativeConfirm(message);
    var timeAfter = new Date();
    if ((timeAfter - timeBefore) < 350) 
    {
        confirmBool = true; 
    }
    return confirmBool;
}
</script>";

            e.Page.ClientScript.RegisterClientScriptBlock(GetType(), "WrapConfirmations", confirmWrapper);
        }

        protected override void OnDeactivated()
        {
            if (IsChrome || IsFirefox)
            {
                WebWindow window = Frame as WebWindow;
                if (window != null)
                {
                    window.CustomRegisterTemplateDependentScripts -= window_CustomRegisterTemplateDependentScripts;
                }
                base.OnDeactivated();
            }
        }
    }
}
{% endcodeblock %}

The controller works by timing the milliseconds to close the confirmation window. If it is less than 350 milliseconds we can assume the confirmation window never opened owing to the checkbox. In this scenario, it returns true (confirm) rather than false (cancel) in order for XAF to function correctly.