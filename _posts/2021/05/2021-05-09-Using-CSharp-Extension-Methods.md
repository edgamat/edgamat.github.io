---
layout: post
title: 'Using C# Extension Methods'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Extension methods in C# are very powerful and I like them a lot. But I have found that I tend to only use them under certain circumstances.

<!--more-->

### What are Extension Methods

In C#, extension methods are static methods that you can use to extend the functionality of a class without having to modify the class itself.

Here is an example. Suppose you have the following code:

```csharp
if (string.IsNullOrWhiteSpace(firstName))
{
    return $"missing_{nameof(firstName)}"
}
```

You can create an extension method to encapsulate the 'missing' calculation:

```csharp
public static class StringExtensions
{
    public static bool IsMissing(this string input)
    {
        return string.IsNullOrWhiteSpace(input);
    }
}
```

The `this` keyword is what makes it possible to expose the new method as if it were part of the class:

```csharp
if (firstName.IsMissing())
{
    return $"missing_{nameof(firstName)}"
}
```

### When to use Extension Methods

For me an extension method feels like the right approach when it will solve a couple of problems. First, it is a scenario when I have code that is reused in a number of places. Second, it needs to be reused in a concise manner where readability is important (like when is it not?).

For example, let's create a function that converts a boolean value to a Yes/No string:

```csharp
public static class StringHelpers
{
    public static string ToYesNo(bool value)
    {
        return value ? "Yes" : "No";
    }
}
```

Without extension methods, using the function looks like this:

```csharp
var viewModel = new CustomerViewModel
{
    FirstName = customer.FirstName,
    LastName = customer.LastName,
    IsActive = StringHelpers.ToYesNo(customer.IsActive)
}
```

Transforming it to an extension method, it now looks like this:

```csharp
var viewModel = new CustomerViewModel
{
    FirstName = customer.FirstName,
    LastName = customer.LastName,
    IsActive = customer.IsActive.ToYesNo()
}
```

To me (a personal preference that may not be the same for everyone) the code using the extension method is easier to read.

### When to Avoid Using Extension Methods

If they are so great, why not use them everywhere when you would want to extend a class? Good question. I typically don't use them when the method alters state or has side effects. Then the extension methods make the code much harder to test or reason about.

Suppose you have a method that queries a database using EF Core:

```csharp
public async Task CompleteOrderAsync(this PosContext context, Order order, CancellationToken token)
{
    var dbOrder = context.Set<Order>()
        .Where(x => Id == Order.Id)
        .FirstOrDefaultAsync(token);

    if (dbOrder == null) throw new OrderNotFoundException(order);

    dbOrder.Status = OrderStatus.Complete;
    dbOrder.StatusDate = DateTime.UtcNow();

    await context.SaveChangesAsync(token);
}
```

Here is an example of it in use:

```csharp
if (order.Shipped)
{
    await context.CompleteOrderAsync(order, token);
}
```

The reason this should probably be avoided is because of the effect it has on testing. Testing any function that calls the `CompleteOrderAsync` method needs to simulate the underlying PosContext which can be difficult in many instances.

You could change the method to work off an interface:

```csharp
{
    public interface IPosContext
    {
        DbSet<TEntity> Set<TEntity>() where TEntity : class;
        
        Task<int> SaveChangesAsync(CancellationToken cancellationToken);
    }
}
```

But even then, you still have to create a simulation (mock/fake) instance of the `IPosContext` any time you want to test a function that calls `CompleteOrderAsync`.

Normally you would expose the method using a class that uses the `PosContext`/`IPosContext`. For example, `OrderService`:

```csharp
public class OrderService: IOrderService
{
    public async Task CompleteOrderAsync(Order order, CancellationToken token)
    {
        ...
    }
}
```

Functions that call this method would have a dependency on `IOrderService`, not `IPosContext`. This is much more desirable because it makes things easier to test (and easier to reason about).

A rather famous example of this in the ASP.NET Core framework is the use of extension methods on the `ILogger` class:

```csharp
logger.LogInformation("test message");
```

`LogInformation` is an extension method. If you need to verify that the call to `LogInformation` was made correctly, you have to verify the underlying call to the `ILogger.Log` method was made:

```csharp
logger.Verify(o => o.Log(
    LogLevel.Information,
    It.IsAny<EventId>(),
    It.Is<FormattedLogValues>(e => e.ToString() == "test message"),
    It.IsAny<Exception>(),
    It.IsAny<Func<object, Exception, string>>()), Times.Once);
```

While I understand the rationale for using extension methods in this case, it does have the side effect of making testing a bit more challenging.

### Summary

Extension method can be a powerful tool, but be careful when to use them. Your test cases will thank you for it.
