---
layout: post
title: 'Managing Entity Framework Core Mappings'
author: 'Matthew Edgar'
pub_date: '2019-10-27'
excerpt_separator: <!--more-->
---

When building a data model using Entity Framework (EF) Core, you can quickly find that the default behavior
is difficult to maintain. Let's look at a better way to handle the large number of mapping data.

<!--more-->

When last we looked at the WarehouseContext class, it looked something like this:

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

All the custom mapping details exist in a single method `OnModelCreating`. Any EF Core project of
significant size will cause this method to grow to an unmanageable size pretty quickly. To fix
this problem, it is better to split the mappings out into a separate file per entity. One might
say, that is what you can do using data annotation attributes, rather than the fluent annotations
in the `OnModelCreating` routine.

## Fluent OnModelCreating versus DataAnnotations

Entity Framework Core uses a set of conventions to build a data model for the classes in the model. For
example, if you have a property in a class called `Id` or `{Class}Id`, EF Core will assume this is
the primary key in the corresponding database table. You can provide additional model configuration
data to override or supplement the convention-based settings.

EF Core provides two ways to provide this additional configuration data. The first is to use data
annotation attributes on the class properties:

```csharp
[Required]
public string Title { get; set; }
```

These data annotations are a set of attributes in the `System.ComponentModel.DataAnnotations` namespace. It
is possible to use multiple attributes on each property:

```csharp
[Required]
[MaxLength(100)]
public string Title { get; set; }
```

The alternative way to provide configuration data is using the fluent API provided by the `ModelBuilder`:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Book>(entity =>
    {
        entity.Property(e => e.Title)
            .IsRequired()
            .HasMaxLength(100);
    });
}
```

In general, the Fluent API approach is considered more powerful since it's configuration data takes
precedence over the DataAnnotations. In addition, there are some configuration settings that are not
possible using DataAnnotations alone. And most importantly, using the Fluent API doesn't require the
Domain model classes to be decorated with their persistence attributes. This is an important aspect
of a Clean Architecture.

Therefore, using the Fluent API to configure the EF Core model is the preferred approach in a Clean
Architecture.

## DbSet Properties (or the lack thereof)

Traditional usage of Entity Framework includes providing a `DbSet` property on the `DbContext` for
each entity in the model. This is one of the ways EF Core determines if an entity is included in the
Data Model. For example:

```csharp
public class WarehouseContext : DbContext
{
    public WarehouseContext(DbContextOptions options) : base(options)
    {
    }

    public DbSet<Book> Books { get; set; }
}
```

In addition, any class discovered as a navigation property on an included class will also
be included in the data model. And lastly, as you would expect, any class explicitly
configured in the `OnModelCreating` method is also included in the data model.

Does that mean the `DbSet` properties are not required? Yes, that is exactly what it means.
If all configuration data is provided via the `OnModelCreating` method, then the `DbSet`
properties are simply there for convenience. The following two statements end up being identical:

```csharp
var book = context.Books.FirstOrDefault(x => x.ISBN == "12345678")

var book = context.Set<Book>.FirstOrDefault(x => x.ISBN == "12345678")
```

Since they are optional, it is something that a Clean Architecture would consider removing. Maintaining
these convenience properties can be a chore and don't provide significant value. It means the `DbContext`
class is kept clean and doesn't grow in size as the number of classes in the data model increases.

## Separation of Configuration Data

Back to the issue at hand: to separate the model configuration data into separate classes. EF Core provides
a means to do this using an interface called `IEntityTypeConfiguration<T>`. Here is an example, based on the `Book`
configuration data:

```csharp
public class BookConfiguration : IEntityTypeConfiguration<Book>
{
    public void Configure(EntityTypeBuilder<Book> builder)
    {
        builder.Property(e => e.ISBN).HasMaxLength(20);
        builder.Property(e => e.Title).HasMaxLength(100);
        builder.Property(e => e.Author).HasMaxLength(50);
    }
}
```

To use this configuration data, you add it to the `DbContext` via the `OnModelCreating` method:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.ApplyConfiguration(new BookConfiguration());
}
```

The `ModelBuilder` also provides a method `ApplyConfigurationFromAssembly` which will use
reflection to find all classes that implement the `IEntityTypeConfiguration<T>` interface
and apply their configurations to the model. This might seem ideal, but it does have some
drawbacks you may wish to consider.

## Which Maps do I Own?

Some situations require a single `DbContext` to be aware of tables it owns and tables it
does not own. For example, let's say the data model needs to access data from an existing set
of tables from a legacy database (you don't own these) as well manage a new set of tables
specific to the application (you own these).

This is a common situation and if you are using EF Core Migrations to manage the shape of the
data model, it is important to only include the tables you own when performing migrations.

One way to achieve this flexibility is to pass in the set of configurations to the
`DbContext`, say via a constructor parameter:

```csharp
private readonly IEntityTypeConfiguration<T>[] _configurations;

public WarehouseContext(IEntityTypeConfiguration<T>[] configurations)
{
    _configurations = configurations;
}
```

But this is not possible. C# won't allow a generic array of T, so a custom solution will be required.

We'll start by creating a non-generic interface:

```csharp
public interface IEntityTypeConfiguration
{
    void Configure(ModelBuilder modelBuilder);
}
```

And then change our configuration class to use this new interface:

```csharp
public class BookConfiguration : IEntityTypeConfiguration
{
    public void Configure(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Book>(builder =>
        {
            builder.Property(e => e.ISBN).HasMaxLength(20);
            builder.Property(e => e.Title).HasMaxLength(100);
            builder.Property(e => e.Author).HasMaxLength(50);
        });
    }
}
```

And now we can use this configuration array in our `DbContext`:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    foreach (var configuration in _configurations)
    {
        configuration.Configure(modelBuilder);
    }
}
```

## What Have We Gained?

With this new ability to pass in the set of configurations, we can separate them into
two groups: configurations for tables we own, and for tables we don't own. If we are creating
a `DbContext` for data migrations, we can choose only the configurations involved in the
migrations:

```csharp
var migrationMappings = new IEntityTypeConfiguration[]
{
    new BookConfiguration()
};

var context = new WarehouseContext(CreateOptions(configuration), migrationMappings);
```

## Summary

We have seen how to configure EF Core using the Fluent API, rather than using
DataAnnotations. This avoids clutter in the Domain Model and keeps it free of persistence
implementation details.
