---
layout: post
title: 'Test Doubles in C# with Moq'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Unit testing relies on creating automated and repeatable tests. A powerful technique supporting this practice is using test doubles. I use the Moq library for this and find it works very well.... with a few tweaks.

<!--more-->

## What are Test Doubles?

The book [XUnit Test Patterns][xunit-patterns] introduces the concept of test doubles as follows:

> Sometimes it is just plain hard to test the **system under test (SUT)** because it depends on other components that cannot be used in the test environment. This could be because they aren't available, they will not return the results needed for the test or because executing them would have undesirable side effects. In other cases, our test strategy requires us to have more control or visibility of the internal behavior of the **SUT**.

> When we are writing a test in which we cannot (or chose not to) use a real **depended-on component (DOC)**, we can replace it with a *Test Double*. The *Test Double* doesn't have to behave exactly like the real **DOC**; it merely has to provide the same API as the real one so that the **SUT** thinks it is the real one!

There are different flavors of Test Doubles as well:

- **Test Stub**: Replace a real component on which the SUT depends so that the test has a control point for the indirect inputs of the SUT.

- **Test Spy**: An observation point for the indirect outputs of the SUT. The Test Spy is a Test Stub with some recording capability.

- **Mock Object**: An observation point that is used to verify the indirect outputs of the SUT as it is exercised. A Mock Object is lot more than just a Test Stub plus assertions; it is used a fundamentally different way.

- **Fake Object**: Replace the functionality of a real DOC in a test for reasons other than verification of indirect inputs and outputs of the SUT.

I find that the Test Stub and Mock Object are used most frequently and will focus on them in this post.

## The Moq Library

[Moq][moq] is a nuget package that contains several classes that support creating test doubles. The library promtes itself as a 'mocking' library, but it is possible to create any type of test double. Here is an example of a Mock Object:

```csharp
interface IAccountingService
{
    bool Register(string accountName)
}

var mockObject = new Mock<IAccountingService>();

mockObject
    .Setup(x => x.Register("userName"))
    .Returns(true);
    
var service = mockObject.Object;

var controller = new AccountingController(service);

var result = controller.RegisterUser("userName");

mockObject
    .Verify(x => x.Register("userName"), Times.Once);
```

And an example of a Test Stub:

```csharp
interface IAccountingRepository
{
    Account GetAccount(string accountName)
}

var dummyAccount = new Account { Name = "test" };

var testStub = new Mock<IAccountingRepository>();

testStub
    .Setup(x => x.GetAccount("test"))
    .Returns(dummyAccount);
    
var repo = mockObject.Object;

var result = repo.GetAccount("test");

Assert.Equal("test", result.Name);
```

## Mock = Test Double

The naming conventions used when writing tests is important to convey intent. So using the word "Stub" in the variable name should imply the intent to be a Test Stub. However, it is confusing the see it formed via a `Mock<T>` class provided by the Moq library. A test stub is not a mock object! To reduce this confusion, I use a sub-class I add to my test library:

```csharp
using Moq;

public class TestDouble<T> : Mock<T> where T: class
{
    public TestDouble()
    {
    }

    public TestDouble(MockBehavior behavior) : base(behavior)
    {
    }
}
```

This allows the tests to be more direct in what they are constructing: Test Doubles.

```csharp
var mockObject = new TestDouble<IAccountingService>();
var testStub = new TestDouble<IAccountingRepository>();
```

You could be a step further and create variation-specific classes (TestStub<T>, MockObject<T>) but I have not needed to resort to that level of detail. However, it could be useful in some context so be mindful of it as an option.

## Summary

Test Doubles are an important tool in constructing automated and repeatable unit tests. The Moq library provides a great means for creating them in C# projects. I few additional steps can make things much clearer when creating the different variations of Test Doubles, not treating them all as 'Mock' objects.

[xunit-patterns]: http://xunitpatterns.com/
[moq]: https://github.com/moq/moq4
