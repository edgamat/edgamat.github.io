---
layout: post
title: 'Too Much of a Good Thing'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Each line of code is a liability. The more code you have, the more can go wrong; the more you have to maintain and test. Finding ways to keep code to a minimum is part art and part science. However, it is a skill that can be learned.

<!--more-->

## The Approach

When writing code, one should always strive to achieve the desired behavior with the least amount of code. This is the science. However, it is possible to take this to the extreme and it will make the code difficult to understand and maintain. This is the art.

You must strike a reasonable balance between minimal code and maintainability. It takes practice and it helps if you have seen a lot of code (both good and bad) to see how this can be achieved. 

Here is an example of a C# function:

```csharp
public static bool CanPlaceOrder(Order order)
{
    if (order != null && order.Amount > decimal.Zero && order.LineItems.Any())
    {
        return true;
    }

    return false;
}
```

This can be reduced by returning the result of the expression:

```csharp
public static bool CanPlaceOrder(Order order)
{
    return order != null && order.Amount > decimal.Zero && order.LineItems.Any();
}
```

It can be reduced a bit further as well:

```csharp
public static bool CanPlaceOrder(Order order) 
    => order != null && order.Amount > decimal.Zero && order.LineItems.Any();
```

It can be debated which of these reductions is easier to maintain. However, the key takeaway is that it can be possible to reduce the amount of code to maintain while still producing good quality code.

Here is another example:

```csharp
var result = (this.Amount > decimal.Zero) ? true : false;
```

This can be reduced to this:

```csharp
var result = this.Amount > decimal.Zero;
```

## Summary

Look for opportunities to reduce the amount of code you maintain. But don't take it to the extreme. Maintainability is also an important trait of your code and striking a balance is a good skill to learn.

