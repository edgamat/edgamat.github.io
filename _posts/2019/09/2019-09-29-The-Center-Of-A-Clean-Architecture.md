---
layout: post
title: 'The Center of a Clean Architecture'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

This is the second in a series of posts exploring the use of Clean Architecture principles to
build an ASP.NET Core REST API application.

In the last post, we created a small .NET Core solution with a class library for the Domain business
logic and a class library for the unit tests covering the business logic.

In this post I want to explore what goes into the Domain business logic. But more specifically, what
doesn't belong.

<!--more-->

One principle of the Clean Architecture is to ensure the Domain business logic is independent
of any frameworks used by your application. So let's take a look at an ASP.NET Core application. It
is dependent on several external NuGet packages to handle processing incoming HTTP requests, including a
host of infrastructure concerns like logging, configuration, dependency injection, security, etc.

We want to ensure that none of that (or as little as possible) appears in the Domain business logic
class library.

## Define the Domain Entities

Recall the idea behind our application, a Book Warehouse:

- The REST API will provide access to the **Warehouse** of books.
- Users will be able to query the Warehouse to determine if a **Book** is in stock.
- Users will be able to create an **Order**.

Let's begin with the most important entity, the **Book**:

```csharp
public class Book
{
    public Guid Id { get; set; }

    public string ISBN { get; set; }

    public string Title { get; set; }

    public string Author { get; set; }

    public decimal Weight { get; set; }
}
```

It represents the intrinsic values of a Book in the real world; and nothing else.

**NOTE**: This will be a naive representation of a book to simplify the design and complexity of
this exploration. Any real-world software application would undoubtedly use a more sophisticated model of
a book.

The next companion class is a **BookShelf**:

```csharp
public class BookShelf
{
    public BookShelf(Book book, int quantity)
    {
        Book = book;
        Quantity = quantity;
    }

    public Book Book { get; set; }

    public int Quantity { get; set; }

    public BookShelf AddBooks(int quantity)
    {
        return new BookShelf(Book, Quantity + quantity);
    }

    public BookShelf RemoveBooks(int quantity)
    {
        return new BookShelf(Book, Quantity - quantity);
    }
}
```

It represents the actual quantity of a particular book the warehouse has in stock. It allows
books to be added to the shelf, and removed.

Let's also define a third entity, the **Warehouse**:

```csharp
public class Warehouse
{
    private readonly IDictionary<Book, BookShelf> _stock = new Dictionary<Book, BookShelf>();

    public string ReceiveBooks(Book book, int quantity)
    {
        if (quantity <= 0) return "invalid_quantity";

        var bookshelf = !_stock.ContainsKey(book)
            ? new BookShelf(book, 0)
            : _stock[book];

        _stock[book] = bookshelf.AddBooks(quantity);

        return string.Empty;
    }
}
```

For now, the warehouse stores a collection of the books in stock and permits books to be received
and placed in the appropriate bookshelf.

None of this code describes how the data will be persisted or how the data will be displayed to a user. Those
concerns are for other layers of the application. The Domain business logic is small in its scope. We will
need to add the other capabilities in such a way as the not let the business logic know how they are accomplished.

For example, notice the validation logic in the Warehouse: `if (quantity <= 0) return "invalid_quantity";`

It doesn't return an English sentence such as "You must provide a quantity greater than zero". That would be
a concern of the User Interface. Perhaps, the application does not use English. Or it might support
multiple languages. Striving to keep these details out of the Domain will keep the code free of such dependencies.

## When to introduce a new Dependency

One area where framework specific dependencies creep into the Domain logic of an application is Configuration. ASP.NET Core applications are no different.

Configuration in ASP.NET Core applications is provided via an `IConfiguration` interface:

```csharp
public class Warehouse
{
    private readonly IConfiguration _configuration;

    public Warehouse(IConfiguration configuration)
    {
        _configuration = configuration;
    }
}
```

You may be tempted to allow your Domain entities to be aware of this interface so that these classes may retrieve
configuration data at run-time. However, that introduces a new dependency.

Introducing a new dependency in .NET applications is possible with a single NuGet command, and this is the problem.
You can add a NuGet package reference several different ways and you can get yourself into trouble quickly.

Most dependencies will come along with their own set of hidden dependencies. So accepting one NuGet package
into your project might make you dependent on several others you didn't know about. Let's say we wanted to
include a reference to the `IConfiguration` interface. It resides in the following NuGet package:

`Microsoft.Extensions.Configuration.Abstractions`

This package is dependent upon the following package:

`Microsoft.Extensions.Primitives`

This package is dependent upon the following packages:

`System.Memory`
`System.Runtime.CompilerServices.Unsafe`

In this example, taking on one dependency has made the Domain logic dependent on
four packages.

Therefore, be careful which dependencies you choose to accept in the Domain logic.You should get into
the habit of asking yourself "Do I need to add this new dependency on my code?", and "What can I do to avoid
adding this new dependency?"

## Avoiding Dependencies

To avoid dependencies, there are two steps used very frequently. First, create a representation in the Domain logic
of functionality you are dependent on. Then, implement the actual logic in some other layer of the application.

In the case of Configuration, instead of being dependent on the `IConfiguration` interface, you may decide to
represent each set of configuration data as separate entities.

```csharp
public class WarehouseConfiguration
{
    public int MaxNumberOfBookshelves { get; set;}
}

public class Warehouse
{
    private readonly WarehouseConfiguration _configuration;

    public Warehouse(WarehouseConfiguration configuration)
    {
        _configuration = configuration;
    }
}
```

It then becomes the responsibility of one of the other layers of the application to convert the
data from the `IConfiguration` representation to the `WarehouseConfiguration` representation. That
new representation is then passed into the Warehouse class.

This technique is very powerful. But it is only made possible through the use of a
dependency injection framework built-in to ASP.NET Core. This framework is part of the Application Services layer
of the application and provides the services necessary to convert the representations of the dependencies
at run-time. I cover [Dependency Injection in another post]({% post_url 2019/08/2019-08-27-Dependency-Injection-In-ASPNET-Core %}) so I will refer you there if you want to look at it
in further detail.

## Summary

To fully explore the concept of what the Domain business logic should contain is what this series of posts will attempt to do:

- Constantly focus on the independence of the Domain business logic.
- Only accept new dependencies after careful consideration.
- Avoid taking on new dependencies wherever possible.
