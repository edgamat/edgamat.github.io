---
layout: post
title: 'The Power of the Compose Method Refactoring'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

The _Compose Method_ Refactoring is one of the more powerful refactorings I use. Let's explore what it is and why it can make such a difference it your code.

<!--more-->

When the COVID-19 pandemic swept through the world's population I, like so many people, was sent to work from home. I left much of my work items at my office in the relative rush to get a workspace setup in a small corner of my house. This week I was able to return to my former office to pick up some items. Mostly I wanted to retrieve some books I had left behind. 

One of those books was _"Refactoring to Patterns"_ by Joshua Kerievsky. The book is a series of refactorings that help move your code towards (or in some cases away from) design patterns. 

The author credits Kent Beck with the origin of the _Compose Method_ refactoring and describes it as a simple refactoring that makes code easier to read and understand. 

### The Pattern

The _Compose Method_ refactoring is meant to move your code towards the _[Composed Method][comp-method]_ Pattern:

> Divide your program into methods that perform one identifiable task. Keep all of the operations in a method at the same level of abstraction. This will naturally result in programs with many small methods, each a few lines long.

For me, there are two important pieces to this pattern. First, you move toward creating smaller methods. It is a learned skill to create smaller methods in my experience, not one that comes naturally. Smaller methods make a lot of sense because it means each of them can be reasoned about (and tested) independently. Smaller methods also are naturally easier for you to change. However, there are limits to this concept (as I will describe in the Benefits and Drawbacks section below).

Second, there is this concept of 'the same level of abstraction'. This too is something that you learn to do over time and doesn't seem to be something I, or other developers, do instinctively. Having a set of methods operate at the same level of abstraction is one of the key aspects of a well-defined API. Whether that be a class's methods and properties or an HTTP endpoint or a user interface, makes no difference. If you can design something in this way, the users of your interface have a much easier time.

I see methods in need of this type of refactoring mostly in JavaScript or TypeScript code. I feel one possible reason for this is the lack of meaningful support for refactorings in the IDE's. It is pretty difficulty for an IDE to perform refactorings in JavaScript. But I have seen my fair share of C# code that could also benefit from this pattern. 

I think Joshua Kerievsky describes a Composed Method best:

> A Composed Method's name communicates _what_ it does, while its body communicates _how_ it does what it does.

### The Refactoring

The refactoring is based primarily on the idea that when you inspect a method, see if there are parts of it that can be extracted out to separate methods. This is the _Extract Method_ pattern in action. You pull parts of the method out into separate methods, ensuring that the name of each of these new methods are kept at a similar level or abstraction. In doing so, the original method calls these new methods in such a way as to describe its logic in a easy to understand way.

#### An Example

Here is a method in C# that doesn't adhere to the _Composed Method_ pattern:

```csharp
public decimal CalculateShippingAmount(Order order)
{
    decimal shippingRate = 0.1M;
    decimal surcharge = 0M;
    
    if (order.Customer.Type == "Gold" ||
        order.Customer.Type == "Platinum" ||
        order.Customer.MemberStartDate < DateTime.Today.AddYears(-2))
    {
        shippingRate = 0.05M;
    }
    
    var totalCost = 0M;
    
    foreach (var item in order.Items)
    {
        totalCost += item.Price * item.Count;
    }
    
    if (totalCost > 99M)
    {
        shippingRate = 0M;
    }
    
    var totalWeight = 0M;
    
    foreach (var item in order.Items)
    {
        totalWeight += item.Weight * item.Count;
    }
    
    if (totalWeight > 25M) 
    {
        surcharge = 10M;
    }
    
    return totalCost * shippingRate + surcharge;
}
```

To begin, I would split the calculations into separate methods. With some well chosen names, the logic becomes much clearer:

```csharp
public decimal CalculateShippingAmount(Order order)
{
    decimal shippingRate = 0.1M;
    decimal surcharge = 0M;
    
    if (IsPreferredCustomer(order.Customer))
    {
        shippingRate = 0.05M;
    }
    
    if (GetTotalCost(order) > 99M)
    {
        shippingRate = 0M;
    }
    
    if (GetTotalWeight(order) > 25M) 
    {
        surcharge = 10M;
    }
    
    return totalCost * shippingRate + surcharge;
}

public bool IsPreferredCustomer(Customer customer)
{
    return customer.Type == "Gold" ||
        customer.Type == "Platinum" ||
        customer.MemberStartDate < DateTime.Today.AddYears(-2))
}

public decimal GetTotalCost(Order order)
{
    var totalCost = 0M;
    
    foreach (var item in order.Items)
    {
        totalCost += item.Price * item.Count;
    }
    
    return totalCost;
}

public decimal GetTotalWeight(Order order)
{
    var totalWeight = 0M;
    
    foreach (var item in order.Items)
    {
        totalWeight += item.Weight * item.Count;
    }
    
    return totalWeight;
}
```

Then, I would move these methods to other classes to better encapsulate the code:

```csharp
public decimal CalculateShippingAmount(Order order)
{
    decimal shippingRate = 0.1M;
    decimal surcharge = 0M;
    
    if (order.Customer.IsPreferredCustomer())
    {
        shippingRate = 0.05M;
    }
    
    if (order.GetTotalCost() > 99M)
    {
        shippingRate = 0M;
    }
    
    if (order.GetTotalWeight() > 25M) 
    {
        surcharge = 10M;
    }
    
    return totalCost * shippingRate + surcharge;
}
```

### Benefits and Drawbacks

The main benefit of this refactoring is the resulting class is much easier to read and understand. It communicates what it does and how it does it. It separates the logic into smaller pieces, which typically allows these methods to be moved to other classes or reused.

There are also two main drawbacks to this refactoring and pattern:

- It can lead to an overabundance of small methods
- It can make debugging more challenging as the logic can be spread over a number of methods.

These are valid concerns. I have seen where using too many smaller methods has made it very challenging to debug a problem or understand the flow of the program. 

I think that the pattern and accompanying refactoring is one that we should use more often, but not every situation requires it. Like all things, sometimes you can have too much of a good thing.

For the most part, I feel we developers could break methods out into smaller, more easy to understand pieces. Imagine a spectrum where large, complicated methods are on one end and smaller, easier to understand methods are at the other end. I humbly suggest we tend to make things that lean towards the large, complicated end than the other. So while I feel the drawbacks of this pattern are valid concerns, we could use a few more Composed Methods in our code.

### Summary

The _Composed Method_ Pattern and _Compose Method_ Refactoring are simple approaches to making your code easier to understand and change. Try them out and see for yourself!

[comp-method]: http://c2.com/ppr/wiki/WikiPagesAboutRefactoring/ComposedMethod.html
