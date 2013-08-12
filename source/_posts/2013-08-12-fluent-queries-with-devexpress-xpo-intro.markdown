---
layout: post
title: "Fluent queries with DevExpress XPO - Intro"
date: 2013-08-12 17:54
comments: true
categories: [c#, devexpress, xpo, xaf]
description: Introduction to fluent queries for DevExpress XPO.
---
There are [many ways to perform queries with XPO](http://documentation.devexpress.com/#xaf/CustomDocument3052).

You can do this:
```c#
    Session.FindObject<Contact>(new BinaryOperator("Name", "Elvis"));
```
or this
```c#
    Session.FindObject<Contact>(CriteriaOperator.Parse("Name = 'Elvis'"));
```
Another way to use the [simplified criteria syntax](http://documentation.devexpress.com/#XPO/CustomDocument2537), and with the [Xpo_EasyFields CodeRush plugin](https://code.google.com/p/dxcorecommunityplugins/wiki/XPO_EasyFields). Then you can do:
```c#
Session.FindObject<Contact>(Customer.Fields.Name == "Elvis");
```
For each of the above, you can optionally query within the transaction by passing in the `PersistentCriteriaEvaluationBehavior.InTransaction` parameter.

Or we can use LINQ via `XPQuery<T>.TransformExpression()`.
```
Session.FindObject<Contact>(
    XPQuery<Contact>.TransformExpression(Session, c => c.Name == "Elvis")
    );
```
All of these methods are powerful, but the power comes at a cost. The syntax is neither elegant nor particularly clear and as a result it is not very practical to maintain or test.

## A Fluent Interface for XPO ##

How about if we could do the following?

```c#
var customer = Session
                .Query()
                  .Contacts
                    .ByName("Elvis");
```
Or, for a more elaborate example:

```c#
var customer = Session
                 .Query()
                 .InTransaction
                    .Contacts
                      .ByPosition("Developer")
                        .ThatHave
                          .NoPhoto()
                        .And
                          .TasksInProgress()
                        .And
                          .TasksWith(Priority.High)            
                 .FirstOrDefault();
```

In the next post I'll show how to put the fluent interface code together.
