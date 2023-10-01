---
layout: post
title: 'Functional C#, No More Nulls'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

What if you decided to remove nulls from C#. What would that look like?

<!--more-->

## Dealing with Missing Values

One advantage that functional programming languages provide is a more formal way of dealing with missing values. In C#, missing values are presented as nulls. This applies to both value types (int, DateTime, custom structs etc.) and reference types (string, custom classes). Let's look at an example.

I have a manager class that returns an entity. But, it is possible that the entity may not exist. The method retrieving the entity must represent the missing entity. There are a few ways this can be accomplished.

### Return a `null` value

Using a `null` as the missing value is the way C# works by default. That is, this mechanism is built into the language. 

```csharp
interface ICustomerManager 
{
    Customer? GetCustomer(string name);
}
```

**NOTE** This assumes you are using C# 8 or later.

It doesn't require any additional code to be written to use the mechanism. However, it does mean that you have more code to actually handle the null values:

```csharp
Customer? customer manager.GetCustomer("john");

if (customer == null) 
{
    // handle null case
}

Console.WriteLine($"{customer.Name}"); // John
```

This can pollute the code with a lot of distractions (boilerplate) to avoid null reference exceptions.

### The Null Object Pattern

A different approach is to return a NullObject (use the [Null Object Pattern](https://en.wikipedia.org/wiki/Null_object_pattern)):

```csharp
interface ICustomerManager 
{
    Customer GetCustomer(string name);
}

class NullCustomer : Customer
{
    public NullCustomer() : 
        base("Unknown Customer", "", "", "", "")
    {
    }
}

Customer customer manager.GetCustomer("unknown");

Console.WriteLine($"{customer.Name}"); // Unknown Customer
```

Using the Null Object Pattern avoids the need for explicit null checking which improve readability. 

However the lack of additional condition checks for null values doesn't come without a cost. You need to create a null object for every entity being used by your code. This can be a lot of additional code to write.

### Use a Wrapper 

Another option is to return the entity in a wrapper class/struct, which indicates the state:

```csharp
interface ICustomerManager 
{
    Option<Customer> GetCustomer(string name);
}

public struct Option<T>
{
    private readonly T? value;

    private readonly bool isSome;

    private bool isNone => !isSome;
}
```

Using a wrapper class is exactly what most functional programming paradigms prefer. And I'd like to explore in more detail. 

## The Option Type

The `Option` type is a type that wraps either a value, or no value. It represents this state as eithre 'None' or 'Some':

- **None** A special value indicating the absense of a value, or
- **Some** A container that wraps the value.

## An Example Using Option&lt;T&gt;

I have chosen a library of functional programming additions for C# based on the material in the "Functional Programming in C#, Second Edition" book by Enrico Buonanno: https://www.manning.com/books/functional-programming-in-c-sharp-second-edition

The library is "LaYumba.Functional" and installed as a Nuget package. I have chosen to include 2 global using statements to help make using the library feel more like it is built into the C# language:

```csharp
global using LaYumba.Functional;
global using static LaYumba.Functional.F;
```

Let's create 2 objects wrapped by the `Option` type:

```csharp
Option<string> nobody = None;
Option<string> john = Some("John");
```

Writing these out to the console we see what we'd expect, due to some convenient overloads on the `Option` type:

```csharp
Console.WriteLine($"{nobody}"); // "None"
Console.WriteLine($"{john}");   // "Some(John)"
```

Interacting with the `Option` type is much easier with pattern matching. In the LaYumba.Functional library, this is implemented using a `Match` function:

```csharp
Option<Customer> customer = CustomerManager.GetCustomer("John");

customer.Match(
    None: () => Console.WriteLine("Unknown customer"),
    Some: (c) => Console.WriteLine(c.Name)
);
```

You can also use the `Match` function to return a value, similar to a switch expression:

```csharp
var name = customer.Match(
    None: () => "Unknown customer",
    Some: (c) => c.Name"
);
```

## Is the Option Type Better?

I suppose using the Option type, and functional programming principles in general, is a subjective decision. In some instances, using the Null Object Pattern has some advantages. And using the built-in `null` concepts in C# are not too difficult when you have null reference types enabled in the later versions of C#.

For example, you can now do this with C# 9 or later:

```csharp
Customer? customer = new("John", "123 Main St", "Anytown", "FL", "12345");

var value = customer switch
{
    null => "Unknown customer",
    _ => customer.Name
};

Console.WriteLine(value); // => "John"
```

So is the `Option` type better than using built-in nulls and pattern matching? I think so:

1. Nullable reference types are a compiler feature. You have to be using a version of the complier that supports them, and you have to enable it properly for it to be effective. 
2. In C#, nullable value types are values wrapped in a structure (Nullable<T>) where nullable reference types are annotations that let the compiler know a value may be null. The `Option` type provides a more consistent approach for handling both nullable value types and nullable reference types. 
3. The `Option` type is one part of the overall functional programming paradigm. It is a building block that more powerful functions can be built upon. 

For example, a switch expression cannot handle functions that return void. The following will not compile:

```csharp
// CS0815: Cannot assign void to an implicitly-typed variable
var _ = c switch
{
    null => Console.WriteLine("Unknown customer"),
    _ => Console.WriteLine(c.Name)
};
```

You would have to introduce wrapper functions to get around this restriction:

```csharp
using Unit = System.ValueTuple;

static Unit PrintValue(string value)
{
    Console.WriteLine(value);
    return default;
}

var _ = c switch
{
    null => PrintValue("Unknown customer"),
    _ => PrintValue(c.Name)
};
```

However, libraries like LaYumba.Functional provide the means to handle actions and functions using the same syntax, which for me is simpler and more readble:

```csharp
customer.Match(
    None: () => Console.WriteLine("Unknown customer"), // action returns void
    Some: (customer) => Console.WriteLine(customer.Name)
);

var name = customer.Match(
    None: () => "Unknown customer", // function returns a string
    Some: (c) => c.Name
);
Console.WriteLine(name);
```

### Summary

I think enabling Null Reference Types in C# projects that support it is a good idea. But we shouldn't stop there. Bring in a functional programming library to make dealing with nulls clearer. And don't shy away from using the Null Object Pattern either. It can be useful as well depending on the complexity of the business logic.
