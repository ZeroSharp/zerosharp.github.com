---
layout: post
title: "Fast batch modifications with DevExpress XPO"
date: 2012-07-24 14:02
comments: true
categories: [c#, devexpress, xpo, performance]
description: An undocumented method of performing fast batch modifications with DevExpress XPO.
---
Last week I wrote about [fast batch deletions](/fast-batch-deletions-with-devexpress-xpo/). In this post I'll show how to do the same for modifications.

Let's assume we want to replace the 'State' property with 'CA' and CostCenter with 123 for all records where the 'City' is 'San Francisco'.  The recommended DevExpress approach would be something like the following:

{% codeblock lang:csharp %}
using (UnitOfWork uow = new UnitOfWork())
{	
	var xpCollection = new XPCollection<MyObject>(uow, CriteriaOperator.Parse("City == 'San Francisco'"));
    foreach (MyObject myObject in xpCollection)
    {
        myObject.State = "CA";
        myObject.CostCenter = 123;
    }
    uow.CommitChanges();
}
{% endcodeblock %}

The problem with the above code is that every record must be loaded and then an individual `UPDATE` command is generated for each modification.  This is necessary for the business logic to be applied correctly (such as the code in methods such as `OnSaving()`). It is also necessary to handle record locking.

If you know that your objects do not require any of this processing, you can use use direct SQL [as described in the XPO documentation](http://documentation.devexpress.com/#XPO/CustomDocument8914). This however requires knowledge of the underlying database table and is not very versatile, (although the `DevExpress.Data.Filtering.CriteriaToWhereClauseHelper()` can help if you choose this route).

However, there is a method similar to the one described in the previous post which is equivalent to the direct SQL approach, but is much easier to use. The approach makes use of an extension method on the `Session` class.

#### Example ####
Since the extension method is somewhat more complicated than for the `Delete` case, I will start by showing an example of use before drilling into the supporting code.

The above example would now look like this:

{% codeblock lang:csharp %}
using (UnitOfWork uow = new UnitOfWork())
{
	uow.Update<MyObject>(
        () => new MyObject(uow) 
                  { 
                     State = "CA", 
                     CostCenter = 123 
                  }, 
        CriteriaOperator.Parse("City == 'San Francisco'"));
}
{% endcodeblock %}

The Update<T> method takes an `Expression<Func<T>>` as the first parameter which allows us to pass in an anonymous type which serves as a template for the modification. This way we get strong typing for the property values.

#### The extensions method ####
Now for the guts of it:

{% codeblock lang:csharp %}
public class PropertyValueStore : List<KeyValuePair<XPMemberInfo, Object>>
{
}

public static class SessionExtensions
{
    public static PropertyValueStore CreatePropertyValueStore(XPClassInfo classInfo, MemberInitExpression memberInitExpression)
    {
        PropertyValueStore propertyValueStore = new PropertyValueStore();

        /// Parse each expression binding within the anonymous class.  
        /// Each binding represents a property assignment within the IXPObject.
        /// Add a KeyValuePair for the corresponding MemberInfo and (invoked) value.
        foreach (var binding in memberInitExpression.Bindings)
        {
            var assignment = binding as MemberAssignment;
            if (binding == null)
            {
                throw new NotImplementedException("All bindings inside the MemberInitExpression are expected to be of type MemberAssignment.");
            }

            // Get the memberInfo corresponding to the property name.
            string memberName = binding.Member.Name;
            XPMemberInfo memberInfo = classInfo.GetMember(memberName);
            if (memberInfo == null)
                throw new ArgumentOutOfRangeException(memberName, String.Format("The member {0} of the {1} class could not be found.", memberName, classInfo.FullName));

            if (!memberInfo.IsPersistent)
                throw new ArgumentException(memberName, String.Format("The member {0} of the {1} class is not persistent.", memberName, classInfo.FullName));

            // Compile and invoke the assignment expression to obtain the contant value to add as a parameter.
            var constant = Expression.Lambda(assignment.Expression, null).Compile().DynamicInvoke();
            
            // Add the 
            propertyValueStore.Add(new KeyValuePair<XPMemberInfo, Object>(memberInfo, constant));
        }
        return propertyValueStore;
    }

    public static ModificationResult Update<T>(this Session session, Expression<Func<T>> evaluator, CriteriaOperator criteria) where T : IXPObject
    {
        if (ReferenceEquals(criteria, null))
            criteria = CriteriaOperator.Parse("True");

        XPClassInfo classInfo = session.GetClassInfo(typeof(T));
        var batchWideData = new BatchWideDataHolder4Modification(session);
        int recordsAffected = (int)session.Evaluate<T>(CriteriaOperator.Parse("Count()"), criteria);

        /// Parse the Expression.
        /// Expect to find a single MemberInitExpression.
        PropertyValueStore propertyValueStore = null;
        int memberInitCount = 1;
        evaluator.Visit<MemberInitExpression>(expression =>
            {
                if (memberInitCount > 1)
                {
                    throw new NotImplementedException("Only a single MemberInitExpression is allowed for the evaluator parameter.");
                }
                memberInitCount++;
                propertyValueStore = CreatePropertyValueStore(classInfo, expression);
                return expression;
            });

        MemberInfoCollection properties = new MemberInfoCollection(classInfo, propertyValueStore.Select(x => x.Key).ToArray());

        List<ModificationStatement> collection = UpdateQueryGenerator.GenerateUpdate(classInfo, properties, criteria, batchWideData);
        foreach (UpdateStatement updateStatement in collection.OfType<UpdateStatement>())
        {
            for (int i = 0; i < updateStatement.Parameters.Count; i++)
            {
                Object value = propertyValueStore[i].Value;
                if (value is IXPObject)
                    updateStatement.Parameters[i].Value = ((IXPObject)(value)).ClassInfo.GetId(value);
                else
                    updateStatement.Parameters[i].Value = value;
            }
            updateStatement.RecordsAffected = recordsAffected;
        }
        return session.DataLayer.ModifyData(collection.ToArray<ModificationStatement>());
    }
}
{% endcodeblock %}

#### Limitations ####
There is currently no way to refer to another field within the assignment expressions - you can only set the value to an `OperandValue`.  So you cannot do

{% codeblock lang:csharp %}
	uow.Update<MyObject>(
        o => new MyObject(uow) 
                  { 
                     // Does not Compile !!!
                     Property1 = o.Property2,
                     // Neither does this !!!
                     Property3 = o.Property3 + 1
                  }, 
        null);
{% endcodeblock %}

In order to fix this, the `evaluator` has to be of type `Expression<Func<T, T>>` instead of `Expression<Func<T>>`, and then you can use expression trees to get an assignment expression. But then there is no way to pass it to a DevExpress `UpdateStatement.Parameter` as an `OperandValue`.

**Update:** The [source code is now available on GitHub](https://github.com/ZeroSharp/XpoBatch).

#### References ####
The code was inspired by [an old blog post Terry Aney](http://www.aneyfamily.com/terryandann/post/2008/04/Batch-Updates-and-Deletes-with-LINQ-to-SQL.aspx) in which he describes a similar approach for LINQ to SQL. 

