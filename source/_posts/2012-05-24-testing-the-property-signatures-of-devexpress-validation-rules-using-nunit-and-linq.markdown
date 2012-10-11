---
layout: post
title: "Testing the property signatures of DevExpress validation rules using NUnit and LINQ"
date: 2012-05-24 16:39
comments: true
categories: [c#, devexpress, nunit, linq, xaf]
description: Meta-tests for XAF validation rules.
---

One of the projects I work on uses the validation module of the [eXpressApp Framework (XAF)](http://devexpress.com/Products/NET/Application_Framework/).  Since the business logic is complex, there are many validation rules defined using the `[RuleFromBoolProperty]`.

One of the recurring problems occurs when the signature of the associated property is incorrect.  Consider the following:

{% codeblock lang:csharp %}
[RuleFromBoolProperty("Invoice_IsAmountGreaterThanZero", 
  DefaultContexts.Save, 
  "Invoice amount must be greater than zero.", 
  UsedProperties = "Amount")]
public bool IsAmountGreaterThanZero
{
    get
    {
        return Amount > 0;
    }
}
{% endcodeblock %}

Notice that the rule is declared `public`.  This causes the getter to be executed when it is not required (see the [note](http://documentation.devexpress.com/#Xaf/clsDevExpressPersistentValidationRuleFromBoolPropertyAttributetopic) in the documentation).  However another problem is that the default behaviour for public properties of XPObjects is to persist them to the datastore which means the application will attempt to create a new column called `IsAmountGreaterThanZero`.

Instead, either property should be declared `protected` or the property should also have the `[NonPersistent]` and `[MemberDesignTimeVisibility(false)]` attributes as well.

Consequently, I wrote the following unit test which will detect any properties which have the `[RuleFromBoolProperty]` attribute.  This is not really a unit test, rather a sort of meta-test

{% codeblock lang:csharp %}
[TestFixture]
public class ValidationRuleDeclarationMetaTests
{
	[Test]
	public void Test_RuleFromBoolPropertyDeclarations_ShouldBeProtectedVisibility()
	{
	    var assemblies = new Assembly[] { typeof(MyObjectAssembly).Assembly };

	    var invalidProperties = assemblies.SelectMany(a => a.GetTypes())
	                                      .SelectMany(t => t.GetProperties(BindingFlags.Public | BindingFlags.Instance))
	                                      .Where(p => p.GetCustomAttributes(typeof(RuleFromBoolPropertyAttribute), true)
	                                                   .Any())
	                                      .Select(p => String.Format("{0}.{1}", p.DeclaringType, p.Name))
	                                      .Distinct();
	    
	    Assert.IsFalse(invalidProperties.Any(), 
	                   "There are 'public' properties with the [RuleFromBoolProperty] attribute. " + 
	                   "These should be 'protected' instead. " + 
	                   "The invalid properties are: " + String.Join(", ", invalidProperties));
	}
}	
{% endcodeblock %}

Now the build will fail whenever a validation property signature is incorrect.


