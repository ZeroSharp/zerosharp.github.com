---
layout: post
title: "Fast batch deletions with DevExpress XPO"
date: 2012-07-12 17:00
comments: true
categories: [c#, devexpress, xpo, performance]
---
When deleting a collection of objects, DevExpress recommends using [`Session.Delete(ICollection objects)`](http://documentation.devexpress.com/#XPO/DevExpressXpoSession_Deletetopic116). This has the same effect as calling the `Delete()` method for every object in the collection so that the business logic is applied correctly. The business logic in this context refers to code such as that in the `OnDeleting()`, `OnDeleted()` methods, but it also includes the clearing of references to the object by other objects. This approach is slow, but ensures the integrity of the data.

If you know that your objects do not require any of this processing, you can use use direct SQL [as described in the XPO documentation](http://documentation.devexpress.com/#XPO/CustomDocument8914). This however requires knowledge of the underlying database table and is not very versatile, (although the `DevExpress.Data.Filtering.CriteriaToWhereClauseHelper()` can help if you choose this route).

An alternative is to use the extension method below:

{% codeblock lang:csharp %}
public static class SessionExtensions
{
    public static ModificationResult Delete<T>(this Session session, CriteriaOperator criteria = null) where T : IXPObject
    {
        if (ReferenceEquals(criteria, null))
            criteria = CriteriaOperator.Parse("True");

        XPClassInfo classInfo = session.GetClassInfo(typeof(T));
        var batchWideData = new BatchWideDataHolder4Modification(session);    
        int recordsAffected = (int)session.Evaluate<T>(CriteriaOperator.Parse("Count()"), criteria);    
        List<ModificationStatement> collection = DeleteQueryGenerator.GenerateDelete(classInfo, criteria, batchWideData);
        foreach (ModificationStatement item in collection)
        {
            item.RecordsAffected = recordsAffected;
        }

        ModificationStatement[] collectionToArray = collection.ToArray<ModificationStatement>();
        ModificationResult result = session.DataLayer.ModifyData(collectionToArray);
        return result;
    }
}
{% endcodeblock %}

Here is an example of how to call the method:

{% codeblock lang:csharp %}
using (UnitOfWork uow = new UnitOfWork())
{
    uow.Delete<MyObject>(CriteriaOperator.Parse("City != 'Chicago'"));
    uow.CommitChanges();
}
{% endcodeblock %}

This achieves the same as result and similar performance to direct SQL, but with cleaner syntax and support for criteria.  Also, since it uses a `ModificationStatement[]`, it works with a remote `IDataStore`.

See the next post for [a similar approach for fast batch modifications](/fast-batch-modifications-with-devexpress-xpo/).

**Update:** The [source code is now available on GitHub](https://github.com/ZeroSharp/XpoBatch).