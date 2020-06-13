---
layout: post
title: 'Setup Order Using Moq'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I use Moq to help create test doubles in my unit tests. I discovered that setting up the order of fake method call can affect how they behave.

<!--more-->

## Moq in a Nutshell

[Moq][moq] is my library of choice when creating test doubles in C# (.NET Core) projects. It is incredibly powerful and straightforward to use. Here's an example of how to create a test stub:

```csharp
var stub = new Mock<IOrderService>();

stub.Setup(s => s.GetOrder(15)).Returns(new Order(15));

IOrderService service = stub.Object;

var order = service.GetOrder(15);

Assert.Equal(15, order.OrderId);
```

This pattern of creating test stubs enables many testing scenarios where you don't want to use the full implementation a class in your test. 

## Generic Input Parameters

An extremely useful feature of Moq is it's support for generic input parameters. Here's the previous example using a generic input parameter. 

```csharp
var stub = new Mock<IOrderService>();

stub
    .Setup(s => s.GetOrder(It.IsAny<int>()))
    .Returns((int orderId) => new Order(orderId));

IOrderService service = stub.Object;

var order = service.GetOrder(15);

Assert.Equal(15, order.OrderId);
```

The `It.IsAny<int>()` parameter is a generic parameter and represents any `int` value. The `Returns` function accepts this parameter and it allows you to create a response using the parameter. This can help make test cases less complicated as a single `Setup` can be reused for different invocations in your test.

## Where Order Matters

I was working on a task recently where I needed to the test stub to behave differently based on the inputs. For a specific set of inputs I wanted the test stub to return `null` so I could test the logic that handled data that wasn't found. When I ran the tests they failed the assertions. I enabled the debugger to see what the test stub was returning. To my surprise, the method call was not returning `null` as I had setup. Why did that occur?

Here's the same scenario using our `IOrderService` test stub from above:

```csharp
var stub = new Mock<IOrderService>();

stub.Setup(s => s.GetOrder(15)).Returns((Order)null);

stub
    .Setup(s => s.GetOrder(It.IsAny<int>()))
    .Returns((int orderId) => new Order(orderId));

IOrderService service = stub.Object;

var order = service.GetOrder(15);

Assert.Null(order);
```

This test fails because `GetOrder(15)` returns a valid Order object, rather than `null`. The order of the `Setup` calls determines which one is used. Since the more generic `Setup` is declared last, it is the one used under execution so the first one that explicitly returns `null` for an orderId = 15 is never used.

One option to correct this issue is to reverse the order of the `Setup` declarations:

```csharp
var stub = new Mock<IOrderService>();

stub
    .Setup(s => s.GetOrder(It.IsAny<int>()))
    .Returns((int orderId) => new Order(orderId));

stub.Setup(s => s.GetOrder(15)).Returns((Order)null);

IOrderService service = stub.Object;

var order = service.GetOrder(15);
Assert.Null(order);

order = service.GetOrder(1);
Assert.NotNull(order);
```

In fact, that is what I did to make my test case work correctly. Unfortunately, it is not always possible to make a change like this. However, the Moq library includes the means to use 'generic' parameters with a bit more precision:

```csharp
stub.Setup(s => s.GetOrder(15)).Returns((Order)null);

stub
    .Setup(s => s.GetOrder(It.Is<int>(orderId => orderId != 15)))
    .Returns((int orderId) => new Order(orderId));
```

The function `It.Is<int>(orderId => orderId != 15)` is a bit different than the `It.IsAny<int>()` function used previously. The `It.Is<T>()` function accepts an expression that filters the inputs with specific values to invoke the specific `Setup` method when being used in a test case.

This approach can work with even more complex inputs:

```csharp
stub
    .Setup(s => s.UpdateOrder(It.Is<Order>(o => o.OrderId == 15)))
    .Returns(true);
```

These 'specific' generic functions (is that a thing?) are much better to use when a test requires different behaviors for a range of input values. Sometimes I get wrapped up in the testing process and forget to take care when declaring `Setup` behaviors to be as precise as possible for the given test. Using the `It.IsAny<T>()` should probably not be my first choice.

## Summary

Using Moq is the way I prefer to create test doubles. It is very flexible in setting up behaviors based on various inputs. But be careful when providing more than one behavior for the same method. The order of the `Setup` descriptions may have an impact on the way `Moq` responds.


[moq]: https://github.com/moq/moq4
