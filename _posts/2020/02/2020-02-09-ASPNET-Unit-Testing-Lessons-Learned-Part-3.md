---
layout: post
title: 'ASPNET Core Unit Testing Lessons Learned Part 3'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When building a Web API endpoint with ASPNET Core, you end up writing a lot of tests. Along the way
I found some ideas that helped me manage them more efficiently. Let's take a look at a few. Note, that
most of these apply to any testing effort, not specifically to ASPNET Core projects.

<!--more-->

## System Under Test

When I start coding a set of tests, I'll group tests by 'system under test', which is the class I want
to test. Even for a small class I can end up with several test methods in a singe test class:

```csharp
public class UserServiceTests
{
    [Fact]
    public void When_Invalid_Input_Service_Throws_Exception()
    {
        // Arrange
        
        // Act
        var sut = new UserService(mockContext.Object)
        
        // Assert
    }

    [Fact]
    public void When_User_Does_Not_Exist_Service_Returns_Null()
    {
        // Arrange
        
        // Act
        var sut = new UserService(mockContext.Object)
        
        // Assert
    }

    [Fact]
    public void When_User_Exists_Service_Returns_Valid_User()
    {
        // Arrange
        
        // Act
        var sut = new UserService(mockContext.Object)
        
        // Assert
    }
}
```

As these classes evolve, I need to maintain these tests. When the constructor changes, I need
to go into each test and make the same change. This can take a lot of time and can introduce
errors pretty easily.

A wiser approach is to use a helper function to create the class being tested:

```csharp
{
    [Fact]
    public void When_User_Exists_Service_Returns_Valid_User()
    {
        // Arrange
        
        // Act
        var sut = CreateSystemUnderTest(mockContext.Object)
        
        // Assert
    }
    
    private UserService CreateSystemUnderTest(DbContext context = null)
    {
        if (context == null)
            context = new Mock<DbContext>();
            
        return new UserService(context)
    }
}
```

The same approach can be used when coding the "Arrange" portion of the test cases. In 
most situations, the arrangement of dependencies and inputs for tests are similar. Create
helper functions for them as well and it can dramatically reduce the amount of code in each test.

## Refactor, Refactor, Refactor

Tests are still code after all, and as such they need to refactored as often as production code does.
The lessons I learned to make with helper functions for maintaining tests were born out of refactoring
out duplication and brittle tests. So I learned to constantly look for was to refactor the code.

Another technique we used to reduce the test code was using xUnit "Theory" attributes rather than "Fact" attributes.
A "Fact" accepts no input parameters, whereas a "Theory" uses input parameters to confirm the behavior
holds true for a variety of inputs. Here is an example:

```csharp
[Fact]
public void When_Date_Is_Before_Days_Return_False()
{
    var input = DateTime.Now.AddDays(1);
    var start = DateTime.Now.AddDays(2);
    var end = DateTime.Now.AddDays(3)
    
    Assert.False(input.IsBetweenDates(start, end)); 
}

[Fact]
public void When_Date_Is_After_Days_Return_False()
{
    var start = DateTime.Now.AddDays(1);
    var end = DateTime.Now.AddDays(2)
    var input = DateTime.Now.AddDays(3);
    
    Assert.False(input.IsBetweenDates(start, end)); 
}

[Fact]
public void When_Date_Is_Between_Days_Return_True()
{
    var start = DateTime.Now.AddDays(1);
    var input = DateTime.Now.AddDays(2);
    var end = DateTime.Now.AddDays(3)
    
    Assert.False(input.IsBetweenDates(start, end)); 
}
```

Using the "Theory" attribute, a test can accept inputs to describe the inputs and expected outputs

```csharp
[Theory]
[InlineData(nameof(TestCases))]
public void IsBetweenDaysReturnsTrue(DateTime input, DateTime start, DateTime end, bool expected)
{
    Assert.Equals(expected, input.IsBetweenDates(start, end));
}

public static IEnumerable<object[]> TestCases()
{
    // When_Date_Is_Before_Days_Return_False
    yield return new object[] { DateTime.Now.AddDays(1), DateTime.Now.AddDays(2), DateTime.Now.AddDays(3), false };
    
    // When_Date_Is_After_Days_Return_False
    yield return new object[] { DateTime.Now.AddDays(3), DateTime.Now.AddDays(1), DateTime.Now.AddDays(2), false };
    
    // When_Date_Is_Between_Days_Return_True
    yield return new object[] { DateTime.Now.AddDays(2), DateTime.Now.AddDays(1), DateTime.Now.AddDays(3), true };
}
```

Even with the comments, this change reduced the code from three functions to two and from 29 lines to 18 lines
of code to maintain. adding a new test requires a new return value from the `TestCases` method. And I would
argue that this new approach is no less readable and understandable than the first approach. 

## Issues and Observations

As a lesson learned, I recommend having helper methods that create the system under test, i.e. the class
whose behavior you are testing. It guards against duplicate setup code and guards against brittle tests
as the constructors of these classes evolve.

When test classes contain multiple test methods, look for opportunities to refactor the code. Just like 
production code, test code can suffer from code smells, duplication, poor naming conventions, and so on.
Keep the test classes neat and tidy and they will continue to provide value as the code evolves.

