---
layout: post
title: "How to improve XAF grid layout for Chrome"
date: 2013-11-08 09:55
comments: true
categories: [c#, devexpress, xaf]
description: The column widths of the DevExpress expressAppFramework list views render poorly when using recent versions of Chrome. Here is a fix.
---
This post proposes a workaround for a specific XAF rendering problem related to recent versions of Google Chrome.

Here is an XAF list view as it looks with IE 10 and DevExpress 13.1.8.

{% img /images/blog/chrome-grid-layout-001.png %}

This is how the same view looks with Chrome 30. Notice how the column widths are rendered poorly. The minimum space is given to the string field and the date and bool fields are too wide.

{% img /images/blog/chrome-grid-layout-002.png %}

What's going on? An update to the Chrome browser occurred in which has caused problems with the rendering of grid views in certain situations. The problem exists in all versions of Chrome great than 25 and also affects the [Google Chrome Frame](http://www.google.com/chromeframe) plugin for IE.

DevExpress were able to fix some scenarios in 13.1.4 and 12.2.11 (such as column sorting), but there are other situations which still pose problems (_e.g._ the fix does not resolve the problem if the _Auto Filter Row_ is enabled, which is the case above.)

Hopefully the Chrome devs will one day fix the root problem. You can keep an eye on [the issue here](https://code.google.com/p/chromium/issues/detail?id=178369)

## The fix ##

Not so much a fix as a workaround. We use a `ViewController<ListView>` to set the grid layout to `UseFixedTableLayout`, but only for Chrome browsers. Something like this:

```c#
/// <summary>
/// Switches to fixed table layout for the main list view when using Chrome or ChromeFrame.
/// </summary>
public class ChromeSpecificListViewController : ViewController<ListView>
{
    protected override void OnViewChanging(View view)
    {
        base.OnViewChanging(view);
        Active["IsChrome"] = IsChrome;
    }

    private static bool IsChrome
    {
        get
        {
            HttpContext context = HttpContext.Current;
            return context != null && context.Request.Browser.Browser == "Chrome";
        }
    }

    protected override void OnViewControlsCreated()
    {
        base.OnViewControlsCreated();

        ASPxGridListEditor listEditor = View.Editor as ASPxGridListEditor;
        if (listEditor == null)
            return;

        ASPxGridView gridView = listEditor.Grid as ASPxGridView;
        if (gridView == null)
            return;

        gridView.Settings.UseFixedTableLayout = true;
    }
}
```    

The resulting view in Chrome:

{% img /images/blog/chrome-grid-layout-003.png %}

That looks better. You may notice the columns will allways be the same width now, which is not quite as good as the IE rendering. As such, it may not be appropriate for all views. But since it's a `ViewController` it's easy to deactivate it by adding an `Active[]` criteria.

#### References

* [Chrome issue](https://code.google.com/p/chromium/issues/detail?id=178369)
* [Support Center Q477066](http://www.devexpress.com/Support/Center/Question/Details/Q477066)
* [Support Center B235945](https://www.devexpress.com/Support/Center/Question/Details/B235945)
* [Support Center Q510468](
https://www.devexpress.com/Support/Center/Question/Details/Q510468)

