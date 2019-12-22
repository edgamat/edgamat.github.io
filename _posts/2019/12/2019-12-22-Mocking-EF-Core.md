---
layout: post
title: 'Mocking EF Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Some would argue that create mock objects for Entity Framework (EF) is no long necessary since
EF Core provides an [In-Memory Database Provider][in-memory]. But even the supporting documentation
is a bit confusing of it's use for unit testing:

> This database provider allows Entity Framework Core to be used with an in-memory database. This can be useful for testing, although the SQLite provider in in-memory mode may be a more appropriate test replacement for relational databases.

<!--more-->

## Test Doubles for EF Core

In the book "[Unit Testing: Principles, Practices, and Patterns][ppp]", Vladimir Khorikov writes:

> A **test double** is an object that looks and behaves like its release intended
> counterpart but is actually a simplified version that reduces the
> complexity and facilitates testing.

When constructing unit tests, I typically look to create test doubles for anything that is going
to slow the tests down, or be difficulty/impossible to control from machine to machine. The EF Core
database context is one such animal, and with EF 6.x, this was always an interesting proposition. Several
mock object libraries provided good support for creating realistic implementations. My favorite has always
been [Moq][moq] and it has served me well.

However when EF Core was released, much attention was placed on the new In-Memory Database Provider since
it would now be possible to skip the complicated process of constructing test doubles. The provider
is used by swapping out the normal builder options with new ones:

```csharp
var options = new DbContextOptionsBuilder<BloggingContext>()
    .UseInMemoryDatabase()
    .Options;
```

## Comparing the In-Memory Database Provider with Mock Objects

There is no denying that swapping out the normal EF Core database provider (e.g. the SQL Server Database Provider)
is not as difficult as constructing a mock implementation. However, it does come with a few drawbacks
that you should be aware of as well.

When using the In-Memory Database Provider, it important to remember that it is not a relational database simulation,
so it can't behave exactly like the real thing. While this might not matter to most unit tests, it can sometimes
give you the wrong feedback. For example, it allows referential integrity constraint violations when saving data.

It is also much slower than running tests using mock test doubles. I have written test suites using xUnit where
each test class that uses the In-Memory Database Provider requires close to 1 second to initialize itself, no
matter how many tests (Fact/Theory) methods are within the class. On a project with potentially dozens
of classes, this is a significant amount of time.

Lastly, and probably most importantly, using the In-Memory Database Provider simulates the _entire_ database,
not only the portions necessary for the test case at hand. When you use mock objects, you need to _explicitly_
create mock versions of each table you are expecting the test to access. Some might argue that this is too
much work and is precisely why the In-Memory Database Provider is a better option. However, when the entire
database is simulated, you may have no way of knowing when the code under test veers off course.

### Stored Procedures

Both the In-Memory Database Provider and mock objects have no support for creating test doubles for stored
procedures. In cases where you require such support within a test case, you will need to simulate them
using virtual execution methods, which is a great topic for another post....

## Creating a Mock DbContext

The Moq library provides great support for mocking objects. However for mocking the DbContext, it
is necessary to include a second library, MockQueryable.Moq.MoqExtensions, that provides a series
of extension methods for just such simulations.

```powershell
dotnet add package Moq
dotnet add package MockQueryable.Moq.MoqExtensions
```

For this example, assume the code under test relies on a DbContext object call `BlogContext` that
includes two objects, `Blog` and `Post`. Use the `Mock` object to create the test double:

```csharp
var mockDbContext = new Mock<BlogContext>();
```

**NOTE**: If the context does not implement a parameter-less constructor, include `null` for each
of these parameters when constructing the test double.

The `DbSet` for each object needs a mock implementation. You can use any collection you wish, but
I typically start with a List<T>:

```csharp
var blogs = new List<Blog>();
```

Then to simulate the `DbSet`, we can use an extension method, `BuildMockDbSet()`, to build the mock:

```csharp
var mockBlogDbSet = blogs.AsQueryable().BuildMockDbSet();
```

Finally attach the mock `DbSet` to the mock context:

```csharp
mockDbContext.Setup(o => o.Set<Blog>()).Returns(mockBlogDbSet.Object);
```

## Supporting Find/FindAsync

If your code depends on the Find/FindAsync methods, they have to be simulated explicitly:

```csharp
mockBlogDbSet
    .Setup(o => o.Find(It.IsAny<object[]>()))
    .Returns((object[] ids) =>
    {
        return mockBlogDbSet.Object.FirstOrDefault(o => o.BlogId.Equals(ids[0]));
    });

mockBlogDbSet
    .Setup(o => o.FindAsync(It.IsAny<object[]>(), It.IsAny<CancellationToken>()))
    .ReturnsAsync((object[] ids, CancellationToken ct) =>
    {
        return mockBlogDbSet.Object.FirstOrDefault(o => o.IBlogIdd.Equals(ids[0]));
    });
```

## Supporting Add/Remove

To support the Add and Remove methods on the mock `DbSet`, use the following simulations:

```csharp
mockBlogDbSet
    .Setup(o => o.Add(It.IsAny<Blog>()))
    .Callback((Blog o) => blogs.Add(o))
    .Returns((EntityEntry<Blog>) null);

mockBlogDbSet
    .Setup(o => o.Remove(It.IsAny<Blog>()))
    .Callback((Blog o) => blogs.Remove(o))
    .Returns((EntityEntry<Blog>) null);
```

## Simplifying Things

Since these simulations for the `Blog` objects will also be necessary for other objects,
it is better to create helper functions using Generics for any data object:

```csharp
public static Mock<DbSet<T>> AddMockDbSetFor<T>(this Mock<CisContext> mockDbContext, IList<T> entities)
    where T : class
{
    var mockDbSet = BuildMockDbSetFor(entities);

    mockDbContext.Setup(o => o.Set<T>()).Returns(mockDbSet.Object);

    return mockDbSet;
}

public static Mock<DbSet<T>> BuildMockDbSetFor<T>(IList<T> entities) where T : class
{
    var mockDbSet = entities.AsQueryable().BuildMockDbSet();

    mockDbSet.Setup(o => o.Add(It.IsAny<T>()))
        .Callback((T o) => entities.Add(o))
        .Returns((EntityEntry<T>) null);

    mockDbSet.Setup(o => o.Remove(It.IsAny<T>()))
        .Callback((T o) => entities.Remove(o))
        .Returns((EntityEntry<T>) null);

    return mockDbSet;
}
```

And adding the Find/FindAsync as well:

```csharp
public static void WithFindSupport<T, TComparable>(this Mock<DbSet<T>> mockDbSet, Func<T, TComparable> prop)
    where T : class
{
    mockDbSet
        .Setup(o => o.Find(It.IsAny<object[]>()))
        .Returns((object[] ids) =>
        {
            return mockDbSet.Object.FirstOrDefault(o => prop(o).Equals(ids[0]));
        });

    mockDbSet.Setup(o => o.FindAsync(It.IsAny<object[]>(), It.IsAny<CancellationToken>()))
        .ReturnsAsync((object[] ids, CancellationToken ct) =>
        {
            return mockDbSet.Object.FirstOrDefault(o => prop(o).Equals(ids[0]));
        });
}
```

Here is how it looks when in use:

```csharp
mockDbContext.AddMockDbSetFor(blogs).WithFindSupport(x => x.BlogId);
```

## Summary

Using the In-Memory Database Provider is a powerful new feature in EF Core. However, it isn't
without it's drawbacks. If you decide to use the traditional mock implementation of a DbContext,
then it too is also supported very well with a few helper functions.

[in-memory]: https://docs.microsoft.com/en-us/ef/core/providers/in-memory
[ppp]: https://www.manning.com/books/unit-testing
[moq]: https://github.com/moq/moq4
