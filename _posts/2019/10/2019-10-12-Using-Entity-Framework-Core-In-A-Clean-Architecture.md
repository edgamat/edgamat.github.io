---
layout: post
title: 'Using Entity Framework Core in a Clean Architecture'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Incorporating persistence in a Clean Architecture is one of the more interesting aspects to consider. Entity
Framework (EF) Core, the object-relational mapper (O/RM) Microsoft provides for .NET Core projects, is familiar
to most developers. Let's explore how to use it with our Clean Architecture where our Domain business logic
is (almost) unaware of it's existence.

<!--more-->

## Handling Persistence in a Clean Architecture

In a Clean Architecture, dependencies go one direction, inwards. Specifically, it is acceptable for the persistence
logic to be aware of the Domain model, but not the other way around. The database is a technical detail that
the application uses to store and retrieve data. However, the Domain shouldn't care how it is done and what
technology is used to implement it. Remember our discussion on taking on dependencies? Well, this is
a dependency your Domain should not accept.

In a typical ASP.NET application, or for that matter most applications, the database is at the heart of
the system. The Domain logic is aware of the persistence implementation details:

UI ---> Domain ---> **Persistence**

In the Clean architecture, we reverse that dependency and ensure the Domain remains at the center of our
architecture:

UI ---> **Domain** <--- Persistence

## Introducing Persistence

To reverse the dependency, we introduce a layer of abstraction to model how our Domain thinks about persistence.
One way (but no matter what people tell you, not the only way) is to use the Repository Pattern. You create an
interface that represents your data persistence requirements. Here is an example:

```csharp
public interface IBookRepository
{
    Book FindByKey(Guid id);

    IEnumerable<Book> All();

    void Insert(Book entity);

    void Update(Book entity);

    void Delete(Guid id);
}
```

This interface is part of the Domain and can be used by Domain classes to store and retrieve Books. However,
the Domain is not where you place the implementation of this interface. The implementation belong in
the Persistence Layer, which we'll use EF Core to create.

## Entity Framework Core

This post assumes you know what EF Core is and how to use it, according to the conventions it provides.

With EF Core, data access is performed using entity classes and a context object that represents a session with the database, allowing you to query and save data. So let's create a new project to store this code:

```powershell
$projectName = "BookWarehouse"

md "$projectName\src\$projectName.Persistence"

# Create the class library for the Persistence logic
cd "$projectName\src\$projectName.Persistence"
dotnet new classlib --no-restore --framework netcoreapp3.0
dotnet add reference "..\$projectName.Domain\$projectName.Domain.csproj"
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

# Add a dependency to the API project
cd "..\$projectName.Api"
dotnet add reference "..\$projectName.Persistence\$projectName.Persistence.csproj"

cd ..\..\..\

# Create the class library for the Persistence layer unit tests
md "$projectName\test\UnitTests.$projectName.Persistence"
cd "$projectName\test\UnitTests.$projectName.Persistence"
dotnet new xunit --no-restore
dotnet add reference "..\..\src\$projectName.Domain\$projectName.Domain.csproj"
dotnet add reference "..\..\src\$projectName.Persistence\$projectName.Persistence.csproj"

# Add a dependency to the API layer's unit tests
cd "..\UnitTests.$projectName.Api"
dotnet add reference "..\..\src\$projectName.Persistence\$projectName.Persistence.csproj"

# Add the new projects to the solution
cd ..\..\
dotnet sln add ".\src\$projectName.Persistence\$projectName.Persistence.csproj"
dotnet sln add ".\test\UnitTests.$projectName.Persistence\UnitTests.$projectName.Persistence.csproj"
```

Notice the new Persistence project. It is not a .NET Standard class library. It is a library based on
the .NET Core framework. This is necessary in order for us to take advantage of the design-time
features of EF Core, namely code-first migrations. And of course, notice that none of these additions
require any changes to the Domain class library. It is unaware that anything has occurred, exactly what
we wanted.

You can use the built-in support to create a database Context:

```csharp
public class WarehouseContext : DbContext
{
    public WarehouseContext(DbContextOptions options) :  base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
    }
}
```

The `DbContextOptions` passed into the constructor describe how to configure the database (and other options) for the context. The `OnModelCreating` method is where you can configure the data model.

Let's recall the `Book` entity from our Domain model:

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

We need to let EF Core know things about the model in order for the persistence to work correctly. Let's say
the string properties have length restrictions:

- ISBN, 20 characters
- Title, 100 characters
- Author, 50 characters

Then our context would look like this:

```csharp
public class WarehouseContext : DbContext
{
    public WarehouseContext(DbContextOptions options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>(entity =>
        {
            entity.Property(e => e.ISBN).HasMaxLength(20);
            entity.Property(e => e.Title).HasMaxLength(100);
            entity.Property(e => e.Author).HasMaxLength(50);
        });
    }
}
```

Using the `OnModelCreating` routine to store all the EF Core data model requirements, we keep
the Domain free of such details.

Now let's look at our implementation of the `IBookRepository`:

```csharp
public class BookRepository : IBookRepository
{
    private readonly WarehouseContext _context;

    public BookRepository(WarehouseContext context)
    {
        _context = context;
    }

    public Book FindByKey(Guid id)
    {
        return _context.Set<Book>().Find(id);
    }

    public IEnumerable<Book> All()
    {
        return _context.Set<Book>().ToList();
    }

    public void Insert(Book entity)
    {
        _context.Set<Book>().Add(entity);
        _context.SaveChanges();
    }

    public void Update(Book entity)
    {
        _context.Set<Book>().Update(entity);
        _context.SaveChanges();
    }

    public void Delete(Guid id)
    {
        var entity = FindByKey(id);
        _context.Set<Book>().Remove(entity);
        _context.SaveChanges();
    }
}
```

**NOTE**: This implementation is perhaps naive and doesn't include a lot of things a real-world application
might require (error handling, code reuse, async access methods, etc.) But just go with it for now. Those details are not critical to the main point which is: _all data access is done outside the Domain layer_.

## Wiring Things Up

To make the application correctly use the new classes, they need to be registered with the Dependency Injection Container. In the ASP.NET Core application, this is performed in the `Startup` class:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var options = CreateOptions(Configuration);
    services.AddScoped(p => new WarehouseContext(options));

    services.AddScoped<IBookRepository, BookRepository>();
}
```

`CreateOptions` is where you would configure whatever mechanism you are using for persistence. This is a
topic beyond the scope of this post, so I'll let that up to you to research further.

Whenever one of our Domain class needs to use the IBookRepository, we pass it in via a constructor parameter:

```csharp
public class Book
{
    private readonly IBookRepository _repository;

    private Book()
    {
    }

    public Book(IBookRepository repository)
    {
        _repository = repository;
    }
}
```

## Almost Unaware

Notice that the entity includes a private parameterless constructor. Entity Framework requires a parameterless constructor in order to materialize objects returned from queries (or loading). This is the only concession made to support persistence
in the Domain layer.

This is a EF Core thing, and other O/RM frameworks may need difference concessions. For example, NHibernate uses
protected parameterless constructors, rather than private ones.

## Summary

We covered a lot of ground here. WE reviewed why we want our Domain to be unaware of the framework used
to manage persistence. We added a new project to our solution to contain the EF Core-related classes. And
we showed how to wire up the DI Container in ASP.NET Core in order for everyone to play nicely with each other.

Next time I'd like to dive a bit deeper into EF Core and how we can manage things a bit easier.
