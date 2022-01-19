---
layout: post
title: 'Decorator Pattern with the Microsoft Dependency Injection Container'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

The Decorator Pattern is a useful pattern in C#. But the built-in DI Container makes using it a bit complicated.

<!--more-->

### The Decorator Pattern

The Decorator Pattern is a design pattern that allows behavior to be added to an object dynamically at runtime. It is useful in cases where you want to add more functionality but don't want to modify the original class definition. Typically this pattern is implemented using wrapper classes that implement the same interface as the existing class. A wrapper class has a reference to an object of the existing class and forwards requests to it while providing the additional functionality. A lot of words, but what does it really look like?

Some typical examples are:

- Adding a cross cutting feature (like caching, logging, or exception handling) without having to pollute the existing class with a bunch of boilerplate code.
- Dynamically add/remove functionality at runtime (like benchmarking).

The implementation I have used is to create a _Decorator_ class that implements the same interface as the existing class and then perform additional functionality before/after forwarding requests to the existing object.

Let's look at an example in C#. I have a class that make a call to a database:

```csharp
public class MyService : IMyService
{
    private readonly MyDbContext _context;
    
    public MyService(MyDbContext context)
    {
        _context = context;
    }
    
    public async Task SubmitOrderAsync(int orderId, DateTime dateSubmitted, CancellationToken token)
    {
        var existingOrder = await _context.Set<Order>()
            .Where(x => x.OrderId == orderId)
            .FirstAsync(token);
        
        existingOrder.Status = "Submitted";
        existingOrder.StatusDate = dateSubmitted;
        
        var _ = await _context.SaveChangesAsync(token);
    }
}
```

The business user has requested a new feature. When an order is submitted, they would like the system to publish an event to the accounting service. One option would be to modify the existing class to be responsible for that behavior. Another option would be to wrap the existing calls to the class with a class that is responsible for that behavior.

Here is what it would look like:

```csharp
public class MyDecoratorService: IMyService
{
    private readonly IMyService _component;
    private readonly IServiceBus _serviceBus;
    
    public MyDecoratorService(IMyService component, IServiceBus serviceBus)
    {
        _component = component;
        _serviceBus = serviceBus;
    }
    
    public async Task SubmitOrderAsync(int orderId, DateTime dateSubmitted, CancellationToken token)
    {
          await _component.SubmitOrderAsync(orderId, dateSubmitted, token);
          
          var orderSubmittedEvent = new OrderSubmittedEvent
          {
              OrderId = orderId,
              DateSubmitted = dateSubmitted
          };
          
          await _serviceBus.PublishAsync(orderSubmittedEvent, token);
    }
}
```

The 'trick' to using this in a .NET project is registering the classes properly with the Dependency Injection (DI) Container. 

### Registering the Decorator Class

The component we are trying to decorate with new functionality is registered as follows:

```csharp
services.AddScoped<IMyService, MyService>();
```

What we want is to replace the current class with the new decorator class instead. But this doesn't do it properly:

```csharp
services.AddScoped<IMyService, MyDecoratorService>();
```

`MyDecoratorService` requires an instance of `IMyService` as a constructor parameter. So what if we included both?

```csharp
services.AddScoped<IMyService, MyService>();
services.AddScoped<IMyService, MyDecoratorService>();
```

The built-in DI container will have two instances of the `IMyService` registered. When a request is made to get an instance of `IMyService`, it will return an instance of the `MyDecoratorService`, since it was the last one registered. So we are no better off.

We need a way to register both simultaneously. Here is how that looks, using the `ActivatorUtilities`:

```csharp
services.AddScoped<IMyService>(p =>
    ActivatorUtilities.CreateInstance<MyDecoratorService>(p, 
        ActivatorUtilities.CreateInstance<MyService>(p, p.GetRequiredService<MyDbContext>()),
        p.GetRequiredService<IServiceBus>()); 
```

The `ActivatorUtilities.CreateInstance` method creates an object using the configured service provider (at runtime). It requires us to define how each of the constructor parameters are assigned. The instance of `MyService` has its `MyDbContext` parameter created from the service provider. The instance of `MyDecoratorService` has its `MyService` parameter created in-line, rather than using the service provider to create it indirectly.

While this works, it can become complex or difficult to maintain depending on how nay classes have decorators or how many constructor parameters are involved.

### Some Other Choices

You can gain some simplicity if you switch to using a third-party container, such as `Autofac`. It has extension methods to make registering decorators simple:


```csharp
var builder = new ContainerBuilder();

// Register the service to be decorated.
builder.RegisterType<MyService>()
       .As<IMyService>();

// Then register the decorator.
builder.RegisterDecorator<MyDecoratorService, IMyService>();
```

Another choice is to use `Scrutor`, which is a set of extensions on top of the built-in DI container:

```csharp
services.AddScoped<IMyService, MyService>();
services.Decorate<IMyService, MyDecoratorService>();
```

Scrutor and Autofac both have some additional features worth checking out:

**Scrutor**  
[https://github.com/khellang/Scrutor](https://github.com/khellang/Scrutor)

**Autofac**  
[https://docs.autofac.org/en/latest/index.html](https://docs.autofac.org/en/latest/index.html)

