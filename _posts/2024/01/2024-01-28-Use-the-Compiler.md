---
layout: post
title: 'Use the Complier to Eliminate Bugs'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Runtime error checking can add a lot of noise to your code. Try using the compiler instead.

<!--more-->

## Let the Compiler Help You

It is a standard practice when coding to verify the input arguments to a function. But this can lead to a lot of additional code and can make things more difficult to read and understand. Rather than checking for errors ad runtime, why not remove the chance of the errors occurring at all?

I spent time last year investigating the Rust programming language. One of its primary reported benefits is its elimination of entire categories of errors due to its design. And in many respects this is true; the compiler prevents you from making decisions that will ead to runtime errors. It catches them at compile time.

I wanted to see what that looks like in C#. Are there ways of relying more on the complier to catch errors?

### Null Reference Exceptions

Starting with C# 10 (.NET 6), the language has provided a means for programs to virtually eliminate null references exceptions, through the use of nullable reference types. Enabling this feature requires you to declare explicitly if a reference type is allowed to be null. While not foolproof, it is vastly more capable of preventing null reference exceptions from occurring. the compiler will not let you get your self into trouble, so to speak. It will return warnings when you don't explicitly check for null values. 

## Case Study: Using EF Core Data

**NOTE** I want to start by stating that the situation I am demonstrating with Entity Framework (EF) Core is not a flaw of EF Core. It is a flaw in the way most developers use the data returned from EF Core queries. In that respect, one could argue it is all about how developers use data, and not about EF Core at all. However, I see this pattern of behavior in my own code mostly related to processing data retrieved from EF Core queries. So I use this case study as a way of highlighting the problem where you are likely to find it.

Let's assume that we have a set of orders for customers stored in a database. We have a process that audits all the orders for a customer, looking for ways to market new products to the customer. The first step is to pull the data out of the database. Orders are classified as Standard (normal pricing) or Special (discounted pricing):

```csharp
public class Order
{
    public Guid Id { get; set; }
    public Guid CustomerId { get; set; }
    public string Description { get; set; } = string.Empty;
    public decimal TotalCost { get; set; }
    public decimal ShippingCost { get; set; }
    public OrderType OrderType { get; set; }
}

var orders = await context.Set<Order>().Where(x => x.CustomerId = customerId).ToArrayAsync()
```

Now we pass it to an audit handler to process:

```csharp
public class OrderHandler
{
    public void AuditCustomerOrders(Order[] orders)
    {
        var standardOrders = orders
            .Where(o => o.OrderType == OrderType.Standard)
            .ToArray();

        ProcessStandardOrders(orders);

        var specialOrders = orders
            .Where(o => o.OrderType == OrderType.Special)
            .ToArray();

        ProcessSpecialOrders(orders);
    }

    private void ProcessStandardOrders(Order[] standardOrders)
    {
        // audit logic goes here
    }

    private void ProcessSpecialOrders(Order[] specialOrders)
    {
        // audit logic goes here
    }
}
```

The AuditCustomerOrders makes an assumption that all the items in the orders argument belong to a single customer. Is it wise for it to make that assumption? In his book _Five Lines of Code_, Christian Clausen writes that any property we do not explicitly check in the code is called an "invariant":

> Properties that we do not explicitly check in the code (or check only with assertions) are called invariants. "This number will never be negative" and "This file definitely exists" are examples of invariants. Unfortunately, it is nearly impossible to ensure that invariants remain valid, especially as the system changes, programmers forget, and new people are added to the team.

_Five Lines of Code_ 2021, Christian Clausen, Page 15

Let's make the invariant go away by adding a validation check:

```csharp
    public void AuditCustomerOrders(Order[] orders)
    {
        if (orders.Select(o => o.CustomerId).Distinct().Count() > 1)
            throw new Exception("Cannot audit orders of different customers");
        ...
```

The same argument can be made of the ProcessStandardOrders and ProcessSpecialOrders. They assume that all input items have the correct OrderType. We should add checks there as well:

```csharp
    private void ProcessStandardOrders(Order[] standardOrders)
    {
        if (standardOrders.Any(x => x.OrderType != OrderType.Special))
            throw new Exception("Cannot audit non-standard orders");

        // audit logic goes here
    }

    private void ProcessSpecialOrders(Order[] specialOrders)
    {
        if (specialOrders.Any(x => x.OrderType != OrderType.Special))
            throw new Exception("Cannot audit non-special orders");

        // audit logic goes here
    }
``` 

While this approach does protect you from violating the invariants, it makes the code more challenging to read, as you add more and more of these checks. A better approach would be to refactor the code to make the compiler aware of the invariants. Then it can tell you (at compile time) when they are being violated. This eliminates the need to clutter the applications with these explicit invariant checks.

Let's tackle the first invariant. The AuditCustomerOrders method needs all items in the orders array to come from the same customer. One way to ensure this invariant is to introduce a new way or representing the orders:

```csharp
public class CustomerOrders(Guid customerId)
{
    private readonly List<Order> _orders = [];

    public Guid CustomerId { get; } = customerId;

    public void LoadOrders(Order[] orders)
    {
        foreach (var order in orders)
        {
            if (order.CustomerId != CustomerId)
                throw new Exception("Cannot add orders for different customers");

            _orders.Add(order);
        }
    }

    public IReadOnlyCollection<CustomerOrder> Orders => _orders.AsReadOnly();
}
```

We can now modify the `AuditCustomerOrders` method to use the new `CustomerOrders` class as its input, removing the need to check for the invariant:

```csharp
    public void AuditCustomerOrders(CustomerOrders orders)
    {
        ...
    }
```

### Was it worth it?

You might ask, what did we accomplish with this refactoring. We added a bunch of additional code to remove an invariant from the code. We originally accomplished the same behavior with just a few lines of code:

```csharp
    if (orders.Select(o => o.CustomerId).Distinct().Count() > 1)
        throw new Exception("Cannot audit orders of different customers");
    ...
```

Was it worth it? I suppose it depends who you ask. 

While the invariant was removed from the `OrderHandler` class, we now have bascially the same invariant check in the `LoadOrders` method of the `CustomerOrders` class. Moving the validation logic to another class is a good separation of concerns. The complexity of the `OrderHandler` class has been reduced. 

This change also prevents a 'leaky abstraction' of sorts. The original input parameter was an array of `Order` objects. This is how the data is extracted from the database. You are allowing that form of the data to influence the design of the `OrderHandler` class. Introducing the `CustomerOrders` class is a way of modelling the data more from a domain-level, than from a persistance-level.

In a way, using the array of `Order` objects is a kind of 'primitive obsession'. We are allowing any array of objects to be used, rather than restricting the set of possible choices to those that all belong to the same customer. 

## Summary

I think the choice of introducing these domain classes to model the data more explicitly is a choice you have to make based on your context. It may not be worth the effort in all situations. But in a lot of cases, I feel it is a powerful technique to reduce the complexity of domain classes making them easier to reason about and test. It gives me feedback mush sooner, in the form of compiler errors when I use the wrong data type for the task at hand.


