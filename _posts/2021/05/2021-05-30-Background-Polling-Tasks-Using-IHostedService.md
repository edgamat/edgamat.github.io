---
layout: post
title: 'Background Polling Tasks Using IHostedService'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Often I encounter processing scenarios that require a background tread to process tasks. In .NET application, using the IHostedService can make it easier.

<!--more-->

### Polling for Changes

A typical example is polling for database changes. The application queries a database to check for new tasks to perform. If no tasks are found, the application sleeps for a specified time interval and then tries again. If tasks are found, then it processes the work and depending on the need will either poll for more changes or sleep before trying again.

Creating a polling service in .NET applications is possible using a hosted service. A hosted service exposes an interface that the Application Host calls during the application startup and shutdown processing. This provides a way for the hosted service to setup resources it needs during startup and to cleanup resources when the host is shutting down.

```csharp
public interface IHostedService
{
    /// <summary>
    /// Triggered when the application host is ready to start the service.
    /// </summary>
    /// <param name="cancellationToken">Indicates that the start process has been aborted.</param>
    Task StartAsync(CancellationToken cancellationToken);

    /// <summary>
    /// Triggered when the application host is performing a graceful shutdown.
    /// </summary>
    /// <param name="cancellationToken">Indicates that the shutdown process should no longer be graceful.</param>
    Task StopAsync(CancellationToken cancellationToken);
}
```

This interface is included in the `Microsoft.Extensions.Hosting` nuget package, which also includes the Generic Host used to create ASP.NET Core applications (or any other type of application) that use dependency injection.

While you could write an implementation of the `IHostedService` yourself, the nuget package includes an abstract base class called `BackgroundService`. It exposes a single method:

```csharp
/// <summary>
/// This method is called when the <see cref="IHostedService"/> starts. The implementation should return a task that represents
/// the lifetime of the long running operation(s) being performed.
/// </summary>
/// <param name="stoppingToken">Triggered when <see cref="IHostedService.StopAsync(CancellationToken)"/> is called.</param>
/// <returns>A <see cref="Task"/> that represents the long running operations.</returns>
Task ExecuteAsync(CancellationToken stoppingToken);
```

### The Design

In this example, the application queries a database table for orders that are ready to ship. If any are found, then a message is published letting other parts of the application know that the orders are ready to ship.

We create a new class based on the `BackgroundService`. The class needs to log activity, so we inject a logger via the constructor. It also needs to read from the database and publish the message so these are injected as well. 

```csharp
public class ReadyToShipPublisher : BackgroundService
{
    private readonly ILogger<ReadyToShipPublisher> _logger;
    private readonly IEventBus _eventBus;
    private readonly IDbContextFactory _contextFactory;

    public ReadyToShipPublisher(ILogger<ReadyToShipPublisher> logger, IEventBus eventBus, IDbContextFactory contextFactory)
    {
        _logger = logger;
        _eventBus = eventBus;
        _contextFactory = contextFactory;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogDebug($"ReadyToShipPublisher is starting.");

        stoppingToken.Register(() =>
            _logger.LogDebug($" ReadyToShipPublisher background task is stopping."));

        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogDebug($"ReadyToShipPublisher task doing background work.");

            await CheckForReadyToShipOrdersAsync(stoppingToken);

            await Task.Delay(5000, stoppingToken);
        }
    }

    public async Task CheckForReadyToShipOrdersAsync(CancellationToken stoppingToken)
    {
        using var context = _contextFactory.GetContext();
        var orders = await context.Set<Order>()
            .Where(x => x.Status == "ReadyToShip")
            .ToListAsync(stoppingToken);

        foreach (var order in orders)
        {
            await _eventBus.PublishAsync(order, stoppingToken);
        }
    }
}
```

The `CheckForReadyToShipOrdersAsync` method is the 'do work here' part of the service. Whatever the work is that needs to be done, this where it goes. The rest is just boilerplate or plumbing code to get you to the point where you can do your work.

The `stoppingToken.Register` delegate lets you know when a request to cancel the operation is being made. This is guaranteed to occur when the application is shutting down, but it can happen at other times as well. 

### The Pattern

This pattern is an important one to both recognize and be capable of implementing. When we need to run a background task, setup a class that implements the `BackgroundService`, write the 'do work' method, including a delay to pause between calls to do more work.

The amount of time the 'pause' should be is very context dependent. Sometimes, it can be a constant amount of time. You can determine the appropriate amount of time by looking at the performance of the system and the business case being modelled. Sometimes, the pause can vary based on the outcome of the 'do work' method or if it the first time the work is being done. For example, one case I have seen is to do work on a maximum number of records at a time. If the maximum is reached and there is more work remaining to do, you might want to adjust the time used to pause processing before picking up more work to do. It really depends on your context. But the best starting point is a constant amount of time. and usually it is best to allow that amount of time to be configured with a runtime parameter.

### Dependency Injection Restrictions

It is critical to understand that classes that inherit from the IHostedService are registered with a 'Singleton' lifetime. That means there is a single scope used by all objects. That includes each loop through the 'do work' cycle. Often it is desirable to create a new scope for all the objects created for a single execution of the 'do work' method.

The way to accomplish this is to inject a service provider into the background service class and create the scope manually:

```csharp
private readonly IServiceProvider _provider;

public ReadyToShipPublisher(IServiceProvider provider)
{
    _provider = provider;
}

while (!stoppingToken.IsCancellationRequested)
{
    _logger.LogDebug($"ReadyToShipPublisher task doing background work.");

    using var scoped = _provider.CreateScope();

    var scopedService = scoped.GetService<IScopedService>()

    await CheckForReadyToShipOrdersAsync(scopedService, stoppingToken);

    await Task.Delay(5000, stoppingToken);
}
```

Without this scope you can often introduce a memory leak or some odd threading issues. Best to avoid those! 
