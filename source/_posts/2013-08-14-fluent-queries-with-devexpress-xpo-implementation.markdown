---
layout: post
title: "Fluent queries with DevExpress XPO - Implementation"
date: 2013-08-14 11:03
comments: true
categories: [c#, devexpress, xpo, xaf]
description: How to implement fluent queries for DevExpress XPO.
---
Continuing from my [last post](/fluent-queries-with-devexpress-xpo-intro), I'll demonstrate how to create a fluent interface so that you can do:

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

First, let's look at the 'beginning' of the fluent interface: the `Query()` extension method.

```c#
public static class QueryExtensions
{
    public static IQueries Query(this Session session)
    {
        return new Queries(session);
    }

    // If we're using XAF, do the same for ObjectSpace as well
    public static IQueries Query(this IObjectSpace objectSpace)
    {
        var xpObjectSpace = objectSpace as XPObjectSpace;
        var session = xpObjectSpace.Session;
        return new Queries(session);
    }
}
```

What does the `Queries()` class look like? 

```c#
    public interface IQueries
    {
        IQueries InTransaction { get; }
        IContactQueries Contacts { get; }
        // One for each queryable object type, e.g.,
        // IDepartmentQueries Departments { get; }       
        // ITaskQueries Tasks { get; }
        // etc.
    }

    public class Queries : IQueries
    {       
        public Queries(Session session)
        {
            _Session = session;
        }

        private readonly Session _Session;
        private bool _InTransaction;

        public IQueries InTransaction
        {
            get
            {
                _InTransaction = true;
                return this;
            }
        }

        private IContactQueries _Contacts;
        public IContactQueries Contacts
        {
            get
            {
                if (_Contacts == null)
                    _Contacts = new ContactQueries(_Session, _InTransaction);
                return _Contacts;
            }
        }
    }
```

If we ignore the `InTransaction` property, it is just a container for the `IContactQueries`. In your application, you would have a similar property for each queryable object type. A new `ContactQueries` instance is created on demand taking into account the whether the `InTransaction` property was visited earlier in the syntax.

Now, let's look at the base classes.

```c#
public interface IQueries<T> : IEnumerable<T>, IFluentInterface
{
}

public class Queries<T> : IQueries<T>
{
    public Queries(Session session, bool inTransaction)
    {
        _Session = session;
        Query = new XPQuery<T>(session, inTransaction);
    }

    private readonly Session _Session;
    protected IQueryable<T> Query { get; set; }

    public IEnumerator<T> GetEnumerator()
    {
        return Query.GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return Query.GetEnumerator();
    }
}
```

So `Queries<T>` wraps an `XPQuery<T>`.

Side note: the inclusion of `IFluentInterface` is a clever trick to improve Intellisense by hiding the `System.Object` members such as `ToString()`. See [Daniel Cazzulino's blog post](http://blogs.clariusconsulting.net/kzu/how-to-hide-system-object-members-from-your-interfaces/).

And now we can implement the `Contact` generic as follows:

```c#
public interface IContactQueries : IQueries<Contact>
{
    IContactQueries ByDepartmentTitle(string departmentTitle);
    IContactQueries ByPosition(string position);
    Contact ByEmail(string email);
}

public class ContactQueries : Queries<Contact>, IContactQueries, IContactThatHaveQueries
{
    public ContactQueries(Session session, bool inTransaction)
        : base(session, inTransaction)
    {
    }

    public IContactQueries ByDepartmentTitle(string department)
    {
        Query = Query.Where(p => p.Department.Title == department);
        return this;
    }

    public IContactQueries ByPosition(string position)
    {
        Query = Query.Where(p => p.Position.Title == position);
        return this;
    }

    public Contact ByEmail(string email)
    {
        return Query.SingleOrDefault(p => p.Email == email);
    }
}    
```    	

There we go. Now we can use our fluent interface:

```c#
var contacts = session.Query().Contacts.ByPosition("Manager");
```

Much more readable. Also more maintainable because all queries are in one place and make use of good old LINQ. It's also easier to test the queries because they are independent of the calling code.

See [a sample implementation](https://github.com/ZeroSharp/Xaf_MainDemo_FluentQueries) built against the DevExpress XAF MainDemo on GitHub.