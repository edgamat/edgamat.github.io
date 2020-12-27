---
layout: post
title: 'Modern C#'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

C# has evolved a lot in recent years and I am guilty of not taking advantage of some of the new syntax to improve my code. Here are some of the new features I need to remember to use more often.

<!--more-->

#### NOTE:

Much of this post was inspired by the talk given by Bill Wagner:

**Change your habits: Modern techniques for modern C#**  
[https://www.youtube.com/watch?v=aUbXGs7YTGo](https://www.youtube.com/watch?v=aUbXGs7YTGo)

### What does it mean to be 'Modern'?

Everything here works with C# Version 7.3, which applies to any app using .NET Framework 4.72 or higher, or .NET Core 2.1 or higher. C# 7.3 was released in May 2018, so it has been around for awhile and I need to use these new features more consistently.

That is not to imply that I should go back through all my applications and rewrite things using the newer syntax. I should only use the new syntax where it makes sense, moving forward. 

From my perspective, using these new features, i.e. being 'modern', is about making your code more clear. The purpose of these new features is not to reduce the lines of code you are writing (although in many cases it does). The intent is to make your code easier to understand and to make your logic more clear to readers. 

### Expression-bodied members

[Expression-bodied members](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members) were introduced in C# 7.0 and allow you to write properties and methods using expressions, rather than the 'get/set' syntax. 

```csharp
// Old way:
class Person
{
    public string FirstName { get; set; }

    public string LastName { get; set; }

    public string GetFullName()
    {
        return $"{FirstName} {LastName}";
    }

    public bool IsValid
    {
        get { return !string.IsNullOrEmpty(FirstName) && !string.IsNullOrEmpty(LastName); }
    }
}

// Using Expression-bodied members:
class Person
{
    public string FirstName { get; set; }

    public string LastName { get; set; }

    public string GetFullName() => $"{FirstName} {LastName}";

    public bool IsValid => !string.IsNullOrEmpty(FirstName) && !string.IsNullOrEmpty(LastName);
}
```

### Throw expressions

[Throw expressions](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/throw#the-throw-expression) are a new way of throwing exceptions in a couple of useful scenarios:

**Conditional Operations**

```csharp
// Old way
public void DisplayFirstPerson(Person[] people)
{
    var first = people.Length > 0 ? people[0] : null;
    if (first == null)
    {
        throw new ArgumentException("No person provided");
    }
}
```

Now you can use the 'throw' as an expression, rather than a statement:

```csharp
// New way
public void DisplayFirstPerson(Person[] people)
{
    var first = people.Length > 0 ? people[0] : throw new ArgumentException("No person provided");
}
```

**Null-coalescing operators**

You can also use a throw exception with the null-coalescing operator:

```csharp
// Old way
private Person _person;

public Person
{
    get
    {
        return _person;
    }
    
    set
    {
        if (value == null)
        {
            throw new ArgumentNullException(nameof(value));
        }
        _person = value;
    }
}

// New way
private Person _person;

public Person
{
    get
    {
        return _person;
    }
    
    set
    {
        _person = value ?? throw new ArgumentNullException(nameof(value));
    }
}
```

### Discard _

[Discards](https://docs.microsoft.com/en-us/dotnet/csharp/discards) allow you to discard the result of a function without having to declare an assignment on the left side of the statement.

Discards are placeholder variables, that are purposely unused in the application code. The discard is designated with a singe underscore (`_`):

```csharp
_ = context.SaveChanges();
```

Discards are interesting when combined with throw expressions, as you can greatly simplify null checks for parameters:

```csharp
if (firstName == null)
{
    throw new ArgumentNullException(nameof(firstName));
}
```

Becomes:

```csharp
_ = firstName ?? throw new ArgumentNullException(nameof(firstName));
```

### Tuples

[Tuples](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples) provide a lightweight syntax for grouping multiple values into a single structure.

```csharp
(decimal, int) result = (100.0M, 5);
Console.WriteLine($"{result.Item1} {result.Item2}");
```

Now you can provide your own fields names:

```csharp
(decimal TotalAmount, int ItemCount) result = (100.0M, 5);
Console.WriteLine($"{result.TotalAmount} {result.ItemCount}");
```

The most obvious use case is for method return values. It provides an elegant means to return more than one value from a method, without having to use output parameters or anonymous types.

```csharp
public (decimal TotalAmount, int ItemCount) SummarizeInvoices(IList<Invoice> invoices)
{
    return (invoices.Sum(x => x.InvoiceAmount), invoices.Count);
}

var totals = SummarizeInvoices(invoices);
```
A lot of times I have resorted to using anonymous types for such things but it becomes very difficult to test such methods. You end up using dynamics to access the result values and it always felt wrong. Tuples provide a much better option.

But there are advantages to using anonymous types versus tuples, so do your research:  
[https://docs.microsoft.com/en-us/dotnet/standard/base-types/choosing-between-anonymous-and-tuple](https://docs.microsoft.com/en-us/dotnet/standard/base-types/choosing-between-anonymous-and-tuple)

One area where using tuples is a problem is return types for ASP.NET endpoints. Tuples don't serialize as you'd expect and can make using them in this scenarios more difficult that anonymous types.

### Pattern Matching

[Pattern matching](https://docs.microsoft.com/en-us/dotnet/csharp/pattern-matching) helps make expressions easier to understand the flow of your code. It applies to the `is` and `switch` expressions.

```csharp
if (entity is Person p)
{
    p.LastModified = DateTime.UtcNow;
}


switch (entity)
{
    case Order o: 
        o.LastModified = DateTime.UtcNow;
        break;

    case Customer c: 
        c.LastModified = DateTime.UtcNow;
        break;

    case Invoice i when i.IsPaid == false: 
        i.LastModified = DateTime.UtcNow;
        break;

    default:
        throw new InvalidOperationException("Unrecognized entity type");
}
```

### TryParse Made Simple

This has been an annoyance of mine for awhile so I am please it has been improved.

```csharp
// Old way
decimal value;

if (decimal.TryParse(input, out value))
{
    result = value * 10;
}

// New way
if (decimal.TryParse(input, out var value))
{
    result = value * 10;
}
```

### Summary

C# has a lot of new features that I need to remember are available and take advantage of. They can make the code I write easier to understand and in most cases do so with fewer lines of code. 
