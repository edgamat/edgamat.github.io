---
layout: post
title: 'Memory Leaks using Dependency Injection with .NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I inherited a .NET Core application that had a memory leak. I found the source to be the way the application used the Dependency Injection framework. It was something I could have easily done myself so I figured I better write down the details to remind my future self to avoid the same issue.

<!--more-->

### Dependency Injection with .NET Core

Dependency Injection (DI) is a set of design patterns that helps produce loosely coupled code. I recommend you read the [Microsoft documentation][dotnet-di] to learn how .NET Core reasons about Dependency Injection.

.NET Core implements three lifetime styles for objects registered with the DI Container:

| Name      | Description                                                                                                        |
| --------- | ------------------------------------------------------------------------------------------------------------------ |
| Transient | Objects are created each time they're requested                                                                    |
| Singleton | Objects are created the first time they're requested and a single instance exists until the application shuts down |
| Scoped    | Objects are created once per scope (e.g. in ASP.NET this is once per request)                                      |

Regardless of whether you are writing an ASP.NET application or another type of application, these same principals apply.

### Using DI with ASP.NET Core

[With ASP.NET Core, Dependency Injection is a first class citizen][aspnet-di]. One registers the classes they wish to inject into controllers, endpoints, views or pages. The DI Container manages the lifetime of these objects and disposes of everything it creates.

You can also take advantage of the DI Container when using [background tasks with hosted services][hosted-services]. Hosted Services in ASP.NET Core follow the same patterns as worker classes in a non web application, say a Console application. The hosted service is registered as a Singleton using the helper method `AddHostedService`:

```csharp
    services.AddHostedService<RecordingService>();
```

### Injecting Services into a Hosted Service

Being registered as a singleton, the hosted service only has one instance in the DI Container. If your service uses polling or some other loop to process work on a schedule, you will need to use the DI Container to create your objects. Here is an example of what I mean:

```csharp
public class RecordingService : BackgroundService
{
    private readonly ILogger _logger;

    private const int PollingDelaySeconds = 15;

    public RecordingService(ILogger<RecordingService> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                await DoWorkAsync(stoppingToken)
            }
            catch (OperationCanceledException ex)
            {
                _logger.LogWarning(ex, "Recording service encountered a timeout");
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Recording service encountered an exception");
            }

            await Task.Delay(TimeSpan.FromSeconds(PollingDelaySeconds), stoppingToken);
        }
    }
}
```

To do anything meaningful, the implementation of `DoWorkAsync` must instantiate other classes. It would be preferable to use the DI Container to do this. 

Let's assume this background service is responsible for recording readings from a set of three sensors (pressure, temperature, and humidity). It reads the current sensor values and then stores the data in a database table using EF Core. The DI Container has the following configuration:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHostedService<RecordingService>();
    services.AddTransient<ISensorAdapter, SensorAdapter>();
    services.AddTransient<IReadingDbContext, ReadingDbContext>();
}
```

To obtain instances of the transient objects in the `DoWorkAsync` method, we could inject them into the service:

```csharp
public class RecordingService : BackgroundService
{
    private readonly ILogger _logger;
    private readonly ISensorAdapter _sensorAdapter;
    private readonly ReadingDbContext _context;

    private const int PollingDelaySeconds = 15;

    public RecordingService(ILogger<RecordingService> logger, ISensorAdapter sensorAdapter, ReadingDbContext context)
    {
        _logger = logger;
        _sensorAdapter = sensorAdapter;
        _context = context;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken) {...}

    private async Task DoWorkAsync(CancellationToken stoppingToken)
    {
        var reading = new Reading
        {
            RecordedAt = DateTime.UtcNow,
            Pressure = _sensorAdapter.GetPressure(),
            Humidity = _sensorAdapter.GetHumidity(),
            Temperature = _sensorAdapter.GetTemperature()
        };

        _context.Set<Reading>().Add(reading);

        var _ = await _context.SaveChangesAsync(stoppingToken);
    }
}
```

No scope is created for a hosted service by default. Therefore the injected services are managed by the root DI container scope, which exists for the lifetime of the application. Even though the `ISensorAdapter` and `IReadingContext` are registered as transient, they effectively have the same lifetime as a singleton. This is called "[Captive Dependency](https://blog.ploeh.dk/2014/06/02/captive-dependency/)" and isn't something we should intend to do. We really should change how the objects are registered with the DI Container. But what if we can't?

### Scoped/Transient Lifetime Objects in a Singleton

It is often necessary to instantiate new (transient) instances of objects each time through a loop. In our example, the two services registered as Transient are exactly that. We don't want to have these instances live a long life. They are designed to be created, used, and disposed of quickly.

To do so, we need to instantiate them manually and not use constructor parameters. 

#### Service Locator Pattern

The Service Locator Pattern is an anti-pattern that exists when a class directly calls the DI Container to obtain objects, rather than allowing the objects to be injected by the container via constructor parameters. The [Microsoft documentation](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection#design-services-for-dependency-injection) explicitly warns against such usage:

> Avoid using the service locator pattern. For example, don't invoke GetService to obtain a service instance when you can use DI

But in the scenario presented here, we don't have much choice. So here is what it might look like:

```csharp
public class RecordingService : BackgroundService
{
    private readonly ILogger _logger;
    private readonly IServiceProvider _serviceProvider;

    private const int PollingDelaySeconds = 15;

    public RecordingService(ILogger<RecordingService> logger, IServiceProvider serviceProvider)
    {
        _logger = logger;
        _serviceProvider = serviceProvider;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken) {...}

    private async Task DoWorkAsync(CancellationToken stoppingToken)
    {
        var sensorAdapter = _serviceProvider.GetService<ISensorAdapter>();
        var context = _serviceProvider.GetService<ReadingDbContext>();

        ...
    }
}
```
The `IServiceProvider` is what we can use to get instances from the DI Container and we can use it explicitly to retrieve the objects we require. 

You may not have noticed, but you just introduced a memory leak. 

### Scoping Services in a Hosted Service

The application I inherited was consuming huge amounts of RAM and in Production would periodically need to be restarted. I was asked to investigate the cause and potentially find a solution. I used the [dotMemory](https://www.jetbrains.com/dotmemory/) program from JetBrains to profile the application as it was running. I discovered the root DI Container was holding onto thousands of instances of a class registered with a transient lifetime scope.

It was following the same pattern I used above:

```csharp
var myInstance = _serviceProvider.GetService<IMyService>()
```

Because hosted services don't have a separate scope, the instance of `IServiceProvider` the DI Container injects is in the root scope of the DI Container. So all objects that are created via `GetService<T>` (or `GetRequiredService<T>`) will be held in memory until the application shuts down. 

And that's the memory leak. 

In my example, every time the `DoWorkAsync` method is called, two new objects are created and managed by the DI Container, but are never removed from memory. In memory, these objects end up in the 'Generation 2' level of the garbage collection process and stay there until the application disposes of the DI Container. Every 15 seconds two new objects are added. Over time, this ends up growing and growing and growing.

When I took a snapshot and inspected the objects in memory, the DI Container object was the parent of 1000's of objects that were disposed, but not released. They weren't released because the DI Container was still active. And it won't be until the application shots down.

So how does one get this to work correctly? Well, the short answer is to [read the documentation](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines#scoped-service-as-singleton) carefully.

In the Microsoft documentation, there is an example of how to implement this process correctly. You need to create a service scope each time through the loop. When the scope is disposed, then all the objects it manages are removed (and the garbage collection process cleans them up correctly). Here is what they recommend:

```csharp
private async Task DoWorkAsync(CancellationToken stoppingToken)
{
    // Create a service scoped
    using (var scope = _provider.CreateScope())
    {
        // Create a scoped provider
        var serviceProvider = scope.ServiceProvider;

        // Use the scoped provider
        var sensorAdapter = serviceProvider.GetService<ISensorAdapter>();
        var context = serviceProvider.GetService<AriesContext>();
        ...
    }
}
```

This is what the ASP.NET Core runtime does for each incoming web request. It creates a scope for the request and all objects created by the DI container are managed within the scope. When the request ends, all these (non-singleton) objects are no longer referenced and can be removed by the garbage collection process.

I made this change to the application and it had the desired effect. The memory leak was gone and the application no longer appeared to need to be restarted periodically. 

But.... that's not the end of the story.

### Asynchronous Operations

There ended up being two places where a scope needed to be added. The second instance was in a loop that included several async/await operations. I placed the new service scope as I had in the previous example:

```csharp
using (var scope = _provider.CreateScope())
{
    var serviceProvider = scope.ServiceProvider;
    foreach (var publisher in publishers)
    {
        var bus = _provider.GetService<IBus>();
        operations.Add(bus.BroadcastAsync(publisher, token));
    }
}
await Task.WhenAll(operations, token);
```

I retested everything and ran the application through the `dotMemory` profiler and it all seemed fine. I deployed the application and immediately upon deployment saw errors occurring on the `WhenAll` line.

In my local environment the `BroadcastAsync` method would run synchronously (they can do that you know...) due to how things were configured and how quickly the work was performed. However, once deployed this operation would be done asynchronously and some of the work was being still being executed after the scope had been disposed. Once this was understood, the fix to was dispose the scope after all the operations had completed.

```csharp
using (var scope = _provider.CreateScope())
{
    var serviceProvider = scope.ServiceProvider;
    foreach (var publisher in publishers)
    {
        var bus = _provider.GetService<IBus>();
        operations.Add(bus.BroadcastAsync(publisher, token));
    }
    
    // Wait for all the operations to complete before disposing of the scope
    await Task.WhenAll(operations, token);
}
```

### Summary

Allowing hosted services to run in a .NET Core application has a number of advantages, including using the Dependency Injection framework for producing loosely couple code. But there are some scenarios that can get you into trouble if you aren't careful. Make sure you don't introduce a memory leak in the process!


[aspnet-di]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
[dotnet-di]: https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection
[hosted-services]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services
