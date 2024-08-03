---
layout: post
title: 'Type Narrowing in .NET 8'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Choosing the incorrect type for data in your applications can be a significnt source of bugs. Let's look how type narrowing can help you. 

<!--more-->

### Types of Data in Programming Languages

Most programming languages provide primative data types to store data. String, numbers, and dates are almost guaranteed to be part of the language:

```csharp
string description = "This is a test";

string status = "Active";

int numberOfDays = 10;

DateTime startDate = new DateTime(2024, 04, 30, 0, 0, 0);
```

Programmers use these data types to form variables, properties and parameters to functions. 

I read the book "Effective TypeScript" by Dan Vanderkam awhile back. He recommends to "Think of Types as Sets of Values": 

A `string` represents all the possible values formed by 0, 1, 2 up to 2,147,483,647 of characters in length.
A `int` represents all possible values from -2,147,483,648 to 2,147,483,647.
A `DateTime` represents all possible instances in time (down to the nanosecond) from 00:00:00.0000000 UTC, January 1, 0001 to 23:59:59.9999999 UTC, December 31, 9999.

For most uses, this represents a near infinite number of possible values in each set.

But our programs typically don't accept all possible values as valid inputs. Programs include validation logic to ensure the values being assigned to properies or passed as parameters to functions are within the smaller subset of valid values:

```csharp
if (description != "")
{
    throw new ArgumentException("Invalid description");
}

if (status != "Active" && status != "Inactive")
{
    throw new ArgumentException("Invalid status");
}

if (numberOfDays <= 0)
{
    throw new ArgumentException("Number of days must be a positive number");
}

if (startDate < DateTime.Today)
{
    throw new ArgumentException("Event cannot start in the past");
}
```

In each of these cases, we have reduced the set of possible values considered valid:

- The set of valid `description` values is all strings except the empty string value
- The set of valid `status` values is only two "Active" and "Inactive"
- The set of valid `numberOfDays` values is only numbers greater than zero
- The set of valid `startDate` values is only from today and in the future

But what happens if you forget to include these validation checks? This is where bugs can creep in.

### Choosing Better Types

When a program includes validation checks on the values of variables and parameters, it is narrowing the possible set of valid values. This is a good thing, as it prevents invalid data from being processed by the program. But it occurs at runtime. You don't know if the values are valid or not at compile time.

Ideally, a program should represent each piece of data using the most narrow type as possible. Relying on the built-in data types makes this next to impossible. 

But that is not to say that programming languages don't provide any assistance. Take C# for example. There are many different built-in numeric data types, each representing a different set of values:

 C# data type | Set of values
 ------------ | -------------
`sbyte` | -128 to 127
`byte` | 0 to 255
`short` | -32,768 to 32,767
`ushort` | 0 to 65,535
`int` | -2,147,483,648 to 2,147,483,647
`uint` | 0 to 4,294,967,295
`long` | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807
`ulong` | 0 to 18,446,744,073,709,551,615

Recent versions of C# include nullable reference types which allow you to decide at compile time if a `string` should or should not consider `null` as a valid value. `DateOnly` represents only the day values (ie. no time component) which is a subset of  all `DateTime` values.

In the case of the `status` value, rather than using a `string` to represent the status, using a `enum` would be limit the possible values to only the valid ones:

```csharp
enum Status
{
    Active,
    Inactive
}
```

It is important to choose the best type for the data at hand. But in most applications, relying only on the built-in 'primitive' types is not enough to reduce or remove all validation checks in the code. Passing `string` values from function to function in the code means you need to perform the same validation checks within each function. It would be wiser to use a type to represent only the valid set of values for a given piece of data, and pass it from function to function. This reduces the number of places where validation must be used making the code simpler and allows the compiler to help you avoid using invalid data.

### Custom Types

In the case of the `description` we could create a custom class to represent non empty strings:

```csharp
using System;

public class NonEmptyString
{
    private readonly string _value;

    public NonEmptyString(string value)
    {
        if (value == "")
        {
            throw new ArgumentException("String value cannot empty", nameof(value));
        }

        _value = value;
    }

    public string Value => _value;
}

NonEmptyString description = new("This is a test");
```

Now it isn't necessary to validate the value in each function to check if it is an empty string. The compiler knows it is not, based on the type we have used to represent it in the code.

Similarly you can create a custom class to represent future dates:

```csharp
using System;

public class FutureDate
{
    private readonly DateTime _date;

    public FutureDate(DateTime date)
    {
        if (date <= DateTime.Today)
        {
            throw new ArgumentException("The date must be in the future.", nameof(date));
        }

        _date = date;
    }

    public DateTime Date => _date;
}

FutureDate startDate = new(new DateTime(2024, 11, 13)) // Today is Aug 3, 2024
```

While these examples are a bit contrived, and not something I would use in a production-ready application, they do demonstrate the point that creating custom types can help make programs less likely to have bugs due to invalid data. It also makes the code much easier to read since the validation logic is isolated into separated areas of the code instead of distributed throughout.

### Summary

Thinking of types as a set of possible values is a powerful concept. It allows you to choose the correct type based on what is considered valid data. But the built-in type in programming languages rarely limit the values enough. Program use validation logic to make up for this shortcoming. Developers should consider using custom types to represent the valid values in order to avoid duplicating validation logic checks throughout the code. this make for easier to read code with fewer bugs to worry about. 
