---
layout: post
title: 'ASPNET Core Unit Testing Lessons Learned Part 4'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When building a Web API endpoint with ASPNET Core, the test cases can get confusing at times, just
like with testing any complex code base. Using fluent assertions can help manage that complexity
and make the tests easier to read and understand. Of course, so does naming things in a logical
manner. Let's take a look at these in more detail.

<!--more-->

## Naming Conventions

Naming things is often described as one of the toughest things to do when programmer. While we have
plenty of evidence to back that up, it doesn't have to be that way for naming unit tests. Use simple
phrases describing the expected behavior. 

Here are some poor examples:

```csharp
[Fact]
public Task DataIsStoredCorrectlyAsync() { }

[Fact]
public Task GetByIdExistsTestAsync() { }

[Fact]
public Task GetDataReturnsDataWhenAuthenticatedAsync() { }
```

Some things to point out: 
- The names use `Async` suffix value, which might be the convention in the production code, but for tests, 
it is meaningless noise which adds nothing to the understanding of what behavior the test is confirming.
- The names use 'Data' which is too generic for any purpose to be useful.
- The names don't use any breaks in the words or proper sentence structure making them difficult to read/understand.

Here are some better examples:

```csharp
[Fact]
public Task When_Save_Called_User_Details_Persisted_Correctly() { }

[Fact]
public Task When_A_User_Exists_User_Can_Be_Retrieved_By_ID()) { }

[Fact]
public Task User_Details_Provided_When_Using_Authenticated_Request() { }
```

In the book "[Unit Testing: Principles, Practices, and Patterns][ppp]", Vladimir Khorikov writes:

> Unit test naming guidelines  
> Adhere to the following guidelines to write expressive, easily readable test names:
> - Don’t follow a rigid naming policy. You simply can’t fit a high-level description of a
> complex behavior into the narrow box of such a policy. Allow freedom of
> expression.
> - Name the test as if you were describing the scenario to a non-programmer who is familiar
> with the problem domain. A domain expert or a business analyst is a good example.
> - Separate words with underscores. Doing so helps improve readability, especially in
> long names.

Good advice.

## Fluent Assertions

Another lesson I learned was the power of using fluent assertions to improve readability. The library a 
developer introduced me to was [Fluent Assertions][fa]. 

The easiest way to demonstrate it's benefit is using an example:

```csharp
string actual = "ABCDEFGHI";

// Option 1
Assert.StartsWith("AB", actual);
Assert.EndsWith("HI", actual);
Assert.Contains("EF", actual);
Assert.Equal(9, actual.Length);


// Option 2
actual.Should().StartWith("AB").And.EndWith("HI").And.Contain("EF").And.HaveLength(9);
```

Option 2 is easier to understand because it uses an intuitive syntax. It flows nicely (in my opinion).

It also returns much better insights when failures occur. If `actual = "BCDEFGH"`, here are the error messages
returned by the two options:

```csharp
// Option 1
Assert.StartsWith() Failure:
Expected: AB
Actual:   BC...

// Option 2
Expected actual to start with "AB", but "BCDEFGH" differs near "BCD" (index 0).
```

You'll note that there are three errors, but only the first one is reported. The Fluent Assertions library
allows you control that behavior with scopes:

```csharp
using (new AssertionScope())
{
    actual.Should().StartWith("AB").And.EndWith("HI").And.Contain("EF").And.HaveLength(9);
}
```

And now all the errors are reported:

```csharp
Expected actual to start with "AB", but "BCDEFGH" differs near "BCD" (index 0).
Expected actual "BCDEFGH" to end with "HI".
Expected actual with length 9, but found string "BCDEFGH" with length 7.
```


## Summary

When naming test cases, a good, easy to understand naming convention is a must. Don't be afraid
to deviate from the conventions used by production code. 

Using Fluent Assertions makes tests much easier to understand and much easier to interpret
errors when they occur.


[ppp]: https://www.manning.com/books/unit-testing
[fa]: https://fluentassertions.com/
[nuget]: https://www.nuget.org/packages/FluentAssertions
