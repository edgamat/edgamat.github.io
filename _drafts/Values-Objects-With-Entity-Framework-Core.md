---
layout: post
title: 'Avoiding Primitive Obsession with Entity Framework Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Primitive Obsession is when primitive types (int, decimal, string, DateTime, etc.) exist in a Domain
Model. Using these data types can cause your domain logic to have a lot of additional code just
to deal with these primitives. You can avoid this by using Value Objects and integrate them with
Entity Framework Core is very straightforward.

<!--more-->

## What are Value Objects

A Value Object is an important concept in Domain-Driven Design, but can be applied in almost
any application, regardless of how it is designed.

- A Value Object represents a quantity or description in the domain.
- It is uniquely identified by the composition of all its property, rather than a key or identifier.
- It is immutable, meaning once created, its internal state cannot change.
- And lastly, it is something that is part of an entity, and not stored on it's own.

Okay, so that's all fine, but what I really need is to see an example, or ten. Let's take
as our first example, money. You can define a money amount in your domain model using the `decimal`
primitive. It satisfies the definition of a Value Object. But you'd be better served to model money
using an explicit `Money` value object.

Another example is an email address. You can use a `string` to represent an email address, but having
an explicit EmailAddress in your model.

Or something more complex like a date range (start, end). Or an address (street, city, state, zip). You can
represent these using Value Objects just as easily as single-value Value Objects.

## Value Object Base Class

There are a few examples of how to represent a Value Object in C#. Here is one from Microsoft:

```csharp
public abstract class ValueObject
{
    protected static bool EqualOperator(ValueObject left, ValueObject right)
    {
        if (ReferenceEquals(left, null) ^ ReferenceEquals(right, null))
        {
            return false;
        }
        return ReferenceEquals(left, null) || left.Equals(right);
    }

    protected static bool NotEqualOperator(ValueObject left, ValueObject right)
    {
        return !(EqualOperator(left, right));
    }

    protected abstract IEnumerable<object> GetAtomicValues();

    public override bool Equals(object obj)
    {
        if (obj == null || obj.GetType() != GetType())
        {
            return false;
        }

        ValueObject other = (ValueObject)obj;
        IEnumerator<object> thisValues = GetAtomicValues().GetEnumerator();
        IEnumerator<object> otherValues = other.GetAtomicValues().GetEnumerator();
        while (thisValues.MoveNext() && otherValues.MoveNext())
        {
            if (ReferenceEquals(thisValues.Current, null) ^
                ReferenceEquals(otherValues.Current, null))
            {
                return false;
            }

            if (thisValues.Current != null &&
                !thisValues.Current.Equals(otherValues.Current))
            {
                return false;
            }
        }
        return !thisValues.MoveNext() && !otherValues.MoveNext();
    }

    public override int GetHashCode()
    {
        return GetAtomicValues()
         .Select(x => x != null ? x.GetHashCode() : 0)
         .Aggregate((x, y) => x ^ y);
    }
}
```

It is part of a [great article][vo-example] describing the use of Value Objects with EF Core, which I have used
to help me learn more about using Domain-Driven Design to build applications.

## Replace Amounts

Here is the `Book` model we have been using:

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

Is the `decimal` type the best option for `Weight`? A `decimal` can be a negative number. Can a book
have a weight less than or equal to zero? Not in our domain, each book must have a weight greater than
zero. The value object can allow us to encapsulate that rule nicely:

```csharp
public class Weight : ValueObject
{
    public decimal Value { get; private set; }

    private Weight(decimal value)
    {
        Value = value;
    }

    public static Weight Create(decimal value)
    {
        if (value <= decimal.Zero)
            throw new ArgumentOutOfRangeException("Weight must be greater than zero.");

        return new Weight(value);
    }

    protected override IEnumerable<object> GetAtomicValues()
    {
        yield return Value;
    }
}
```

Now the `Book` property becomes:

```csharp
public Weight Weight { get; set; }
```

## Persisting Value Objects

Now we need to let EF Core know how to store this value. There are several approaches:

- Use the built-in `ValueConverter` model and use an explicit conversion
- Add a set of implicit operators on the value object to handle the conversions

For the first approach, we can modify the model mapping using the `HasConversion` configuration:

```csharp
modelBuilder
    .Entity<Book>()
    .Property(e => e.Weight)
    .HasConversion(
        v => v.Value,
        v => Weight.Create(v));
```

You can also declare an explicit class for the conversion, which is helpful if you need to share the conversion
in multiple places:

```csharp
var converter = new ValueConverter<Weight, decimal>(
    v => v.Value,
    v => Weight.Create(v));

modelBuilder
    .Entity<Book>()
    .Property(e => e.Weight)
    .HasConversion(converter);
```

## Implicit Conversions

Instead of explicitly using `v.Value` and `Weight.Create()` to indicate the conversion, you can
add implicit operators to your Value Object to make things simpler:

```csharp
public static implicit operator decimal(Weight value)
{
    return value.Value;
}

public static explicit operator Weight(decimal value)
{
    return Weight.Create(value);
}
```

Now the converter can be written as:

```csharp
var converter = new ValueConverter<Weight, decimal>(
    v => (decimal)v,
    v => Weight(v));
```

### Why Do This?

The implicit operators will make your Domain logic easier to read, since you can change things like this:

```csharp
// Without implicit operators
Weight totalWeight = Weight.Create(book.Weight.Value * quantity);

// With implicit operators
Weight totalWeight = book.Weight.Value * quantity;
```

You can evan improve things further by using an implicit multiplication operator:

```csharp
public static Weight operator *(Amount a, decimal b)
{
    return Weight.Create(a.Value * b);
}
```

Now the code is even simpler:

```csharp
// With implicit operators
Weight totalWeight = book.Weight * quantity;
```

## Summary

Value Objects are an important part of Domain-Driven Design and can replace primitive data types
in your domain model. This makes your code easier to read and helps encapsulate business rules
in an easy to maintain manner.

And EF Core supports this by allowing you to declare value conversions when persisting the
model to the database.

I encourage you to explore this more as you look to make a transition to a richer domain model
for your business application.

NOTE: An alternative ValueObject base class has been written by Vladimir Khorikov and is available
via a NuGet package:

```
Install-Package CSharpFunctionalExtensions
```

[vo-example]: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects
