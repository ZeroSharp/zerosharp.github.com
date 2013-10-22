---
layout: post
title: "Three ways to store a list of currency codes in XAF"
date: 2013-10-22 18:15
comments: true
categories: [c#, devexpress, xaf]
description: Three solutions to a storing a list of related objects in DevExpress XAF.
---
In the last post we looked at three solutions to a relatively simple XAF requirement. In this post I'll discuss another XAF challenge explain the options and provide a versatile and maintainable solution.

In my DevExpress XAF application, I have an object which has several properties like this:

{% img /images/blog/currency-list-editor-002.png %}

In each case, the field is a comma-separated list of currency codes. These fields are not very important to the model - they are used mainly for reporting.

Let's look at 3 different ways of handling these fields.

## Option 1 - Use a plain old string field ##

The _lightest_ option would be just to declare them as a normal XPO string field:

```c#
private string _List1Currencies;
public string List1Currencies
{
    get
    {
        return _List1Currencies;
    }
    set
    {
        SetPropertyValue("List1Currencies", ref _List1Currencies, value);
    }
}
```
It's certainly simple and maintainable, but it's not very user-friendly. There is no fancy interface to help with the input.  We can perhaps improve things slightly by providing edit masks and/or validation rules to check the input, but careful typing is the only way to change the values.

## Option 2 - Declare an association property ##

The _heaviest_ option is to declare each such property as a many-to-many relationship.

```c#
public class Container: XPObject {
    public Container(Session session) : base(session) { }

	public string Name;

    [Association("List1Currencies")]
    public XPCollection<Currency> List1 { 
        get { return GetCollection<Currency>("List1"); }
    }
}

public class Currency: XPObject {
    public Currency(Session session) : base(session) { }

    [Size(3)]
    public string Code;

	public string Name;

    [Association("List1Currencies")]
    public XPCollection<Container> List1Container { 
        get { return GetCollection<Container>("List1Container"); }
    }
}
```

This works great - we get a nice interface for selecting the currencies and the end result looks like this:

{% img /images/blog/currency-list-editor-003.png %}

However, it's quite a heavy solution for something quite simple. For each such relationship XPO will generate a new intermediate table. If we look at the database schema, we see the following:

{% img /images/blog/currency-list-editor-004.png %}

And in the model there are two new views.

{% img /images/blog/currency-list-editor-005.png %}

If we have 5 such properties, we end up with 5 intermediary tables and 10 new views.

Now, depending on your requirements that may be acceptable. If those relationships are important to your model, then the overhead may be justified. In my situation, these are minor fields and I do not want to burden the model or the database with extra complexity if I can avoid it.

## Option 3 - Create a custom property editor ##

With the help of [the documentation](http://documentation.devexpress.com/xaf/CustomDocument3097.aspx) an old Support Center issues, I was able to quite quickly put together a custom editor which gives the end user a nice interface while keeping it simple. The bulk of the logic is in the `SerializedListPropertyEditor` base class (see the end of the article for the link to the code), but the principle is as follows:

Create a new subclass:

```c#
    [PropertyEditor(typeof(String), false)]
    public class CurrencyListPropertyEditor : SerializedListPropertyEditor<Currency>
    {
        public CurrencyListPropertyEditor(Type objectType, IModelMemberViewItem info)
            : base(objectType, info) { }

        protected override string GetDisplayText(Currency currency)
        {
            return String.Format("{0}\t{1}", currency.Code, currency.Name);
        }

        protected override string GetValue(Currency currency)
        {
            return currency.Code;
        }
    }
```

Then decorate each property with 

```c#
private string _List1Currencies;
[ModelDefault("PropertyEditorType", "Solution1.Module.Web.CurrencyListPropertyEditor")]
public string List1Currencies
{
    get
    {
        return _List1Currencies;
    }
    set
    {
        SetPropertyValue("List1Currencies", ref _List1Currencies, value);
    }
}
```

Now the user gets a pretty editor to select the currencies, but the field is just a string field.

{% img /images/blog/currency-list-editor-001.png %}

The editor supports use of the `[DataSourceProperty]` and `[DataSourceCriteria]` properties too, so you can easily filter the collection.

It is easy to provide a similar editor for any object type - just create a subclass of `SerializedListPropertyEditor<T>` where `T` is your persistent type.

You can download [a working sample project on GitHub](https://github.com/ZeroSharp/Xaf_CurrencyListPropertyEditor).