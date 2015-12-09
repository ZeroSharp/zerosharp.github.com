---
layout: post
title: "DevExpress 2015.2 review part 2"
date: 2015-12-09 14:43
comments: true
categories: [c#, devexpress, xaf]
description: A review of the new DevExpress 2015.2 release with particular focus on the XAF ASP.NET batch editing in list views.
---
This is the second and final dive into some of the new DevExpress XAF 2015.2 features. The [first part](/devexpress-15-dot-2-review-part-1/) covers the Report Designer and the new XML serialisation.  

## Batch editing ##
Another feature I'm excited about is the support for batch editing within the web application grids. 

Let's see what happens when combined with the validation rules. What happens if I edit two rows but only one has a validation problem - does the whole batch get rejected? Or just the row with the problem?

First I modified the edit mode of the Tasks grid to `Batch` via the model.

{% img /images/blog/devexpress-15-2-review-001.png %}

Then I added a new `RuleRequiredField` validation rule via the model.

{% img /images/blog/devexpress-15-2-review-002.png %}

I started up the MainDemo application and navigated to the Tasks view and tried to delete the subject of multiple rows at the same time.

{% img /images/blog/devexpress-15-2-review-003.png %}

I never even got to press Save because of another new 15.2 feature: [inplace validation](https://community.devexpress.com/blogs/eaf/archive/2015/11/24/xaf-validation-module-enhancements-for-windows-and-the-web-coming-soon-in-v15-2.aspx)! The rules are being validated _without_ a round trip to the server!

So let's try another way. _Inplace Validation_ does not work with all rule types, so I deleted my `RuleRequiredField` from the model and instead added a new `RuleFromBoolProperty` to the `DemoTask` object as follows:

```c#
[MemberDesignTimeVisibility(false)]
[RuleFromBoolProperty("SubjectIsRequired", 
    DefaultContexts.Save, 
    "Subject is required.", 
    UsedProperties = "Subject", 
    SkipNullOrEmptyValues = false)]
public bool IsSubjectRequired
{
    get
    {
        return !String.IsNullOrWhiteSpace(Subject);
    }
}
```

Now we go back to the list view for Task and try to set multiple subjects to empty.

{% img /images/blog/devexpress-15-2-review-004.png %}

Fantastic! This time the broken rules appear for each row and it is quite clear which message belongs to which object.

So the batch editing makes use of inplace validation when it can, but handles more complex validation rules well too. This is an excellent combination because the inpalce validation will help to make the client seem very quick and responsive.

## Conclusion ##

This concludes my review of DevExpress 2015.2. We've looked in some detail at two of the most impressive new features - the way reports are serialised and the improvements to client-side validation. These changes help with performance and ease of maintenance and I'm very happy to see that the DevExpress team has focused on these areas. Looking forward to the 2016 releases!



