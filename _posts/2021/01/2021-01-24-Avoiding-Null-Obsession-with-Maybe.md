---
layout: post
title: 'Avoiding Null Obsession with Maybe in C#'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In the previous blog post I wrote about using a _Result_ class. In this post I want to explore a companion called _Maybe_. 

<!--more-->

### The Over-use of Null

C# is obsessed with `null`. No doubt about it. A lot of attention is made throughout the code we write to handle null values. If an instance contains null and we don't handle this case, the code throws the dreaded `NullReferenceException`. So you end up with a lot of code that looks like this:

```csharp
var input = Console.ReadLine();
if (input != null)
{
    var dayOfWeek = Enum.Parse<DayOfWeek>(input);
}
```

Or this:

```csharp
public InsertCustomer(Customer data)
{
    if (data == null)
    {
        throw ArgumentNullException(nameof(data))
    }
    
    ...
}
```

It would be nice if we could deal with nulls a little more elegantly. 

### Introducing Maybe

_Maybe_ is a structure in C# that is a wrapper around a value that may contain a value, or may not. In the book "[Functional Programming in C#][book]" by Enrico Buonanno, this concept is called `Option`. But I prefer `Maybe` because it doesn't conflict with other uses of `Option` in .NET Core.

The interface of the `Maybe` structure is:

```csharp
public struct Maybe<T> where T : class
{
    private T _value;
    public T Value => _value;

    public bool HasValue => _value != null;

    public Maybe(T value)
    {
        _value = value;
    }
}
```

And here is an example of the `Maybe` concept in use:

```csharp
Maybe<Customer> customer = handler.GetCustomer(id);
if (!customer.HasValue)
{
    return NotFound()
}

return Ok(customer.Value);
```

But is this really superior to using `null`? Perhaps not. The real benefits of `Maybe` appear with some additional operations and helper classes.

The first modification is to add the negative check for a value:

```csharp
public bool HasValue => _value != null;
public bool HasNoValue => !HasValue;
```

Now you can write this instead:

```csharp
Maybe<Customer> customer = handler.GetCustomer(id);
if (customer.HasNoValue)
{
    return NotFound()
}

return Ok(customer.Value);
```

Much cleaner in my opinion.

### Adding Operators

In it's current form, here is how I can use the `Maybe` concept in a method:

```csharp
public Maybe<Customer> GetCustomer(int id)
{
    if (id == 0)
    {
        return new Maybe<Customer>(null);
    }

    Customer data = ...

    return new Maybe<Customer>(data);
}
```

It would be nice if we could do this:

```csharp
public Maybe<Customer> GetCustomer(int id)
{
    if (id == 0)
    {
        return null;
    }

    Customer data = ...

    return data;
}
```

This reduces the syntax changes when using the `Maybe` structure. This can be accomplished by introducing operators:

```csharp
public static implicit operator Maybe<T>(T value)
{
    return new Maybe<T>(value);
}

public static bool operator ==(Maybe<T> maybe, T value)
{
    if (maybe.HasNoValue)
        return false;

    return maybe.Value.Equals(value);
}

public static bool operator !=(Maybe<T> maybe, T value)
{
    return !(maybe == value);
}
```

Now we can move onto more elegant changes.

### A Step Towards Functional Programming

As demonstrated with the `Result` class, you can add a 'Match' method to run functions against the value of `Maybe`:

```csharp
public R Match<R>(Func<T, R> hasValue, Func<R> hasNoValue)
{
    return HasValue
        ? hasValue(_value)
        : hasNoValue();
}
```

This can reduce a controller action to this:

```csharp
public IActionResult Get(int id)
{
    return handler.GetCustomer(1)
        .Match(
            value => Ok(value) as IActionResult,
            () => NotFound()
        );
}
```

You can also introduce a mapping function, similar to the `Select` in LINQ:

```csharp

public static class MaybeExtensions
{
    public static Maybe<R> Map<T, R>(this Maybe<T> optT, Func<T, R> f)
        where T : class
        where R : class
        => optT.Match(
            t => f(t),
            () => default);
}

...

public IActionResult Get(int id)
{
    return handler.GetCustomer(1)
        .Map(x => new CustomerViewModel(x))
        .Match(
            value => Ok(value) as IActionResult,
            () => NotFound()
        );
}
```

Notice now that we don't have to throw exceptions if a value is `null`. The code is capable of handling this case in a clear and concise manner.

### Summary

Littering code with a lot of `null` checks can significantly reduce your code's readability. Finding alternatives like `Maybe` can make things better. 

[book]: https://www.manning.com/books/functional-programming-in-c-sharp
