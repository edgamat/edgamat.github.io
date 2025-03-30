---
layout: post
title: 'Background Service Health Checks in .NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Many (most) of the applications I write include one or more background services. Monitoring their status is challenging. Let's see how health checks can help.

<!--more-->

### Health Checks in ASP.NET

ASP.NET includes support for health checks and we use it a lot for ensuring the application is healthy. We include checks for a lot of things:

 - Is the database up and running?
 - Have all the data migrations ran successfully (using EF Core)?
 - Is the Rabbit MQ instance up and running?
 - Can we connect to a remote API?

A great explanation of the Health Checks features is found here:

[https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks)

Registering the health checks is done at startup:

```csharp
// Program.cs

builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy())

...

app.UseHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = JsonResponseWriter.WriteResponse
});
```

Check out this link for a description of the `JsonResponseWriter` class:

[Using ASP.NET Core Health Checks]({% post_url 2021/02/2021-02-21-Using-ASPNET-Core-HealthChecks %})

### Background Services

A typical background service is a class that inherits from the `BackgroundService` class and implements a method where processing is done:

```csharp
public class PollingService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await DoWorkAsync(stoppingToken);

            await Task.Delay(5000, stoppingToken);
        }
    }

    private Task DoWorkAsync(CancellationToken stoppingToken)
    {
        // Do Work Here
        return Task.CompletedTask;
    }
}
```

I want to have a health check in place to let me know when a background service isn't in a healthy state. What is a healthy state? It depends on the work being done, of course. Let's start by reporting the health of a background service based on how long it has been since it performed any work.

Let's add a member to record that last time work was completed successfully:

```csharp
private DateTime _successfullyCompletedAt = DateTime.MinValue;
```

Then we can use it to check on the status:

```csharp
while (!stoppingToken.IsCancellationRequested)
{
    try
    {
        await DoWorkAsync(stoppingToken);

        _successfullyCompletedAt = DateTime.UtcNow;
    }
    catch (Exception)
    {
        // Log Exception
    }

    if (_successfullyCompletedAt < DateTime.UtcNow.AddSeconds(90))
    {
        // Service is not healthy!
    }

    await Task.Delay(5000, stoppingToken);
}
```

In this example, I chose 90 seconds as the threshold for the service to be considered unhealthy. You will need to determine the appropriate threshold for your service, taking into account how long does doing work typically take. Typically, set the threshold to be a bit longer than the range the work usually takes to get processed.

With this in place, we can now check the status of the service. If the time it takes to process work takes too long or if exceptions occur while processing the work, it is considered to be unhealthy.

### Gather Health Check Data

In order for a health check to report the status of the background service, we need to establish a mechanism to make the successfully completed at value to the health check. To do that, we will introduce a class to encapsulate that data:

```csharp
public class BackgroundServiceHealth<T> where T : BackgroundService
{
    private DateTime _successfullyCompletedAt = DateTime.MinValue;

    public DateTime LastSuccessfullyCompletedAt => _successfullyCompletedAt;

    public void SuccessfullyCompleted() => _successfullyCompletedAt = DateTime.UtcNow;
}
```

Then we replace the private member in the background service with the new class, injected via the constructor:

```csharp
private readonly BackgroundServiceHealth<PollingService> _serviceHealth;

public PollingService(BackgroundServiceHealth<PollingService> serviceHealth)
{
    _serviceHealth = serviceHealth;
}

protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        try
        {
            await DoWorkAsync(stoppingToken);

            _serviceHealth.SuccessfullyCompleted();
        }
        ...
    }
}
```

The `BackgroundServiceHealth` instances are registered as singletons:

```csharp
// Program.cs
builder.Services.AddSingleton(typeof(BackgroundServiceHealth<>));
```

### The Background Service Health Check

Here is the health check class:

```csharp
internal class BackgroundServiceHealthCheck<T> : IHealthCheck where T : BackgroundService
{
    private readonly BackgroundServiceHealth<T> _serviceHealth;

    public BackgroundServiceHealthCheck(BackgroundServiceHealth<T> serviceHealth)
    {
        _serviceHealth = serviceHealth;
    }

    public Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        var lastSuccessfullyCompletedAt = _serviceHealth.LastSuccessfullyCompletedAt;

        var timeAgo = DateTime.UtcNow.Subtract(lastSuccessfullyCompletedAt);

        var data = new Dictionary<string, object> {
            { "Last successfully completed at", lastSuccessfullyCompletedAt.ToString("s") },
            { "Time ago", timeAgo.ToString("c") }
        };

        if (lastSuccessfullyCompletedAt > DateTime.UtcNow.AddSeconds(-15))
        {
            return Task.FromResult(HealthCheckResult.Healthy("Completing Work Successfully", data));
        }

        return Task.FromResult(HealthCheckResult.Unhealthy("Not Completing Work Successfully", null, data));
    }
}
```

It is registered in the DI Container:

```csharp
builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy())
    .AddCheck<BackgroundServiceHealthCheck<PollingService>>(nameof(PollingService));
```

The health check will return the status as part of the health check endpoint response:

```json
{
    "status": "Healthy",
    "results": {
        "self": {
            "status": "Healthy",
            "description": null,
            "data": {}
        },
        "PollingService": {
            "status": "Healthy",
            "description": "Completing Work Successfully",
            "data": {
                "Last successfully completed at": "2024-03-17T10:07:00",
                "Time ago": "00:00:01.0568311"
            }
        }
    }
}
```

### Additional Considerations

In this example, I registered the background service and the `BackgroundServiceHealth` data as singletons. You may need to change how these work if they are registered differently. 

In addition, the `BackgroundServiceHealth` is not currently thread-safe. If that is a requirement you will need to modify the `BackgroundServiceHealth` to only allow one thread to update the `_successfullyCompletedAt` value at a time.

### Summary

Monitoring the health of background services is important to include in your health checking process. I've shown a simple example here. Hopefully it is enough of a starting point for you to include it in ASP.NET projects.

**NOTE** Inspiration for this post came from:

Worker services in Kubernetes with health checks  
[https://www.johanohlin.com/posts/2019-11-07-worker-service-in-kubernetes-with-health-checks/](https://www.johanohlin.com/posts/2019-11-07-worker-service-in-kubernetes-with-health-checks/)
