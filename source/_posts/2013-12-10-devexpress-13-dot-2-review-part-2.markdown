---
layout: post
title: "DevExpress 13.2 Review - Part 2"
date: 2013-12-10 16:10
comments: true
categories: [c#, devexpress, xaf]
description: Part 2 of a review of the new DevExpress 13.2 release with particular focus on XAF soft validation.
---
This is the second part of a review of the new DevExpress 13.2. In [the last part](/devexpress-13-dot-2-review-part-1) we looked in-depth at the new [Reports V2](). In this part I'll go over some of the other new features including the support for warnings and confirmations in the validation module.

## Soft Validation Rules ##
With 13.2, DevExpress adds support for warning/confirmation messages to the validation engine. Warnings can be used to handle an unusual but valid data entry. An example would be:

{% blockquote %}
The date of birth results in an age of over 100. Are you sure?
{% endblockquote %}

Here the age of the contact is unusual but not impossible, so instead of prohibiting it entirely, we ask the user for confirmation before saving.

Let's add this rule to the MainDemo. Open the model and navigate to the Validation node. Add a `RuleValueComparison` and configure it as follows

{% img /images/blog/devexpress-13-2-review-010.png %}

Of course you could instead also define the same rule with an attribute on the `Birthday` property. Something like:

```c#
[RuleValueComparison("IsOlderThan100_Warning", 
    DefaultContexts.Save,
    ValueComparisonType.GreaterThan, "AddYears(Now(), -100)",
    "Birthday makes this Contact older than 100. Are you sure?",
    ParametersMode.Expression,
    ResultType = ValidationResultType.Warning)]
```

Notice the new `ResultType` parameter is set to `ValidationResultType.Warning`.

Another typical use is to provide better handling of duplicates. Consider the following:

```c#
[RuleCombinationOfPropertiesIsUnique("DuplicateName_Warning", 
    DefaultContexts.Save, 
    "LastName;FirstName", 
    "There is already a Contact with the name {FullName}. Are you sure?", 
    ResultType = ValidationResultType.Warning)]
public class Contact : Person {
  //etc ...
```

And then this is what happens if I try to add another John Nilsen.

{% img /images/blog/devexpress-13-2-review-011.png %}

Another scenario would be to warn the user of something which needs attention. Such as "Warning! You are running out of funds in your current account." Or "Warning! Deleting this record will format your hard drive."

### List Views ###

Soft validation also works in the list views, even with multiple selection, but there are a couple of things that don't work very smoothly yet and I would expect the web implementation to evolve over the coming releases. 

In order to demonstrate this I need to use a context which allows multiple selection such as deletion. So let's decorate our class with the following simple rule.

```c#
    [RuleCriteria("Deletion_Warning", DefaultContexts.Delete, "1=0", "Warning! Are you sure?", ResultType = ValidationResultType.Warning)]
```

Then I select all the contacts and press `Delete`, after the confirmation window, I get this:

{% img /images/blog/devexpress-13-2-review-014.png %}

### Web Application ###

Soft validation is also available in the web application. This is what a warning looks like.

{% img /images/blog/devexpress-13-2-review-015.png %}

I would prefer to see a _Confirm_ button rather than the current _Ignore_ checkbox since a button requires a single click to validate.

When there are several warnings and errors at the same time, the current implementation displays them all. I think it would be preferable if warnings were not displayed unless there are no errors. Unless DevExpress provide this as an option soon, I will attempt to extend the controller in this regard in a future post.

## Other new features ##
In the 13.2 release, there is now support for runtime extension of the model. DevExpress is calling this feature _custom fields_ and (again) it is marked as beta. This is not a feature I've looked at, but there are a few other non-XAF DevExpress novelties which I'd like to see working within XAF. These include [new themes](https://www.devexpress.com/Subscriptions/New-2013-v2-Beta.xml?product=aspnet#ctl00_ctl00_Content_Content_ctl659) and [support for grid batch editing](https://www.devexpress.com/Subscriptions/New-2013-v2-Beta.xml?product=aspnet#ctl00_ctl00_Content_Content_ctl660).

## One warning ##
{% img left /images/blog/devexpress-13-2-review-001.png %}
{% img right /images/blog/devexpress-13-2-review-002.png %}
The default directories for the installation have changed again. I'm sure DevExpress has some good reason for this, but if, like me, you have several different versions installed you end up with a confusing directory tree. Whenever this happens I always end up having to modify build scripts and config files so that all my tools work as expected. For those of you who use Red Gate's _.NET Reflector_, you can find my config file [in this gist](https://gist.github.com/shamp00/7748150).

## Conclusions ##
For the 13.2 release, DevExpress have focused on making the existing functionality work better rather than developing new modules. 

Better reports. Better validation. A better framework all round.
