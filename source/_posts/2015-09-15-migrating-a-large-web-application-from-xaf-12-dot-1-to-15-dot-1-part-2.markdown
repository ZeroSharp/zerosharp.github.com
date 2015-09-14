---
layout: post
title: "Migrating a large web application from XAF 12.1 to 15.1 - Part 2"
date: 2015-09-15 10:37
comments: true
categories: [c#, devexpress, xaf]
description: Describing some of the challenges encountered in upgrading to the latest DevExpress expressAppFramework. In this part I tackle the hardest migration problem - the XAF Validator.
---
This is the second part of a [series](/migrating-a-large-web-application-from-xaf-12-dot-1-to-15-dot-1-part-1) about migrating a large application from XAF 12.1 to XAF 15.1.

In the 13.1 release, DevExpress made a change to the way XAF `Validator` class is used. It now requires an `IObjectSpace` parameter corresponding to the object. It is needed to correctly evaluate any rules which are descendants of the `RuleSearchObjectProperties`. These include:

- `RuleCombinationOfPropertiesIsUnique`
- `RuleIsReferenced`
- `RuleObjectExists`
- `RuleUniqueValue`

A lot of our code has been around for years now and the older parts rely heavily on `Session` and `UnitOfWork` instead of `IObjectSpace` For the most part our application used `IObjectSpace` only within `ViewControllers`. 

But there were several situations where we need the validator where we don't have an `IObjectSpace`. For instance we sometimes need to validate from within [method actions](https://documentation.devexpress.com/#eXpressAppFramework/clsDevExpressPersistentBaseActionAttributetopic) (decorated with the `ActionAttribute`). For performance reasons, we pass criteria to our middleware and it uses a `UnitOfWork` to run the method on each object. So in this case, there was no `IObjectSpace` to pass to the XAF `Validator`.

{% pullquote %}
Here I had a refactoring dilemma to solve. Either I need to rewrite all of the affected rules so that they no longer make use of the `IObjectSpace`. For instance, I could use `RuleFromBoolProperty` instead. In our application, this would mean rewriting about 50 rules. Or alternatively, I could go through the entire code base looking for `new UnitOfWork()` and `new Session()` and try to use an `IObjectSpace` instead. {"When writing code I often find myself having to decide between the 'quick' fix and the 'right' fix."} Here, moving to `IObjectSpace` throughout is clearly the right fix and although it would take more time to implement, the system will be more in-line with best XAF practices throughout.
{% endpullquote %}

 Eventually, the refactoring was complete and all unit tests are passing. I was eager to run a multi-user load stress test against the 15.1 version to compare performance under load. I have described in previous posts how to [stress test XAF applications](/load-testing-xaf-overview/). I'll be sharing the results in the next post.
