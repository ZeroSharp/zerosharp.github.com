---
layout: post
title: "Better handling of Decimals in the XAF audit trail"
date: 2013-01-15 10:29
comments: true
categories: [c#, devexpress, xaf]
---
The following screenshot shows the detail view of an object change from the DevExpress XAF Audit Trail. The `DecimalValue` property was changed from 123.45 to 543.22.

{% img /images/blog/audit-trail-001.jpg %}

Why is the `OldValue` property is displayed with two trailing zeros? The corresponding property is defined as follows:

{% codeblock lang:csharp %}

[DefaultClassOptions]
public class MyClass : XPObject
{
    public MyClass(Session session)
        : base(session)
    { }

    //...

    private decimal _DecimalValue;
    [ModelDefault("DisplayFormat", "{0:n2}")]
    public decimal DecimalValue
    {
        get
        {
            return _DecimalValue;
        }
        set
        {
            SetPropertyValue("DecimalValue", ref _DecimalValue, value);
        }
    }

    private XPCollection<AuditDataItemPersistent> _ChangeHistory;
    public XPCollection<AuditDataItemPersistent> ChangeHistory
    {
        get
        {
            if (_ChangeHistory == null)
            {
                _ChangeHistory = AuditedObjectWeakReference.GetAuditTrail(Session, this);
            }
            return _ChangeHistory;
        }
    }
}

{% endcodeblock %}

### Explanation and fix ###
A C# `decimal` is a type which represents a number's value *and its precision*. It actually stores the number of trailing zeros along with the value. For the `NewValue`, it has stored the decimal value as the user entered it - with no trailing zeros. Howevever, for the `OldValue`, it has retrieved the value from the database and used the SQL column definition to determine the precision.

The default SQL column type that XPO column type for properties of type `decimal` is the `money` type ([see the MSDN documentation](http://msdn.microsoft.com/en-us/library/aa933242.aspx)) which stores 4 decimal places of precision. If we override this with, say, a `DECIMAL(28, 13)`, the audit trail would show 13 decimal places of precision.

From the user's perspective, this looks a little confusing, so let's fix it.

During the initialization of your application (in Application_Start for a web application), add an event to the `AuditTrailService` as follows.

    AuditTrailConfig.Initialize();

and then declare the `AuditTrailConfig` helper class as follows:

{% codeblock lang:csharp %}

public static class AuditTrailConfig
{
    public static void Initialize()
    {
        AuditTrailService.Instance.SaveAuditTrailData += Instance_SaveAuditTrailData;
    }

    static void Instance_SaveAuditTrailData(object sender, SaveAuditTrailDataEventArgs e)
    {
        NormalizeOldValuesDecimalPrecision(e);
    }

    private static void NormalizeOldValuesDecimalPrecision(SaveAuditTrailDataEventArgs e)
    {
        var decimalAuditTrailDataItems = e.AuditTrailDataItems
                                          .Where(i => i.OldValue is decimal);

        foreach (AuditDataItem auditTrailItem in decimalAuditTrailDataItems)
        {
            // remove any trailing zeros from OldValue
            auditTrailItem.OldValue = ((decimal)auditTrailItem.OldValue).Normalize();
        }
    }
}

{% endcodeblock %}

The `Normalize()` method is an extension method. See the [trick in my last post](/how-to-remove-the-trailing-zeros-of-precision-from-a-c-number-decimal/) for more information.

{% codeblock lang:csharp %}

public static class DecimalExtensions
{
    public static decimal Normalize(this decimal value)
    {
        return value / 1.000000000000000000000000000000000m;
    }
}

{% endcodeblock %}

And then the same change would be logged as follows.

{% img /images/blog/audit-trail-002.jpg %}