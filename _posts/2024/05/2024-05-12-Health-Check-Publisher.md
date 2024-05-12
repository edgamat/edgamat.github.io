---
layout: post
title: 'Publishing health Check Data in C#'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Sometimes pulling health check data is not possible. Can we push the data?

<!--more-->

### Background

Some application are not publicly accessible from the Internet. This makes it difficult to use an external monitoring solution to check on the health of a system. Monitoring tools typically will send an HTTP request to a health check endpoint to determine if the application is healthy. But if it can't access the website, then that option is not available.

What we could do instead is to push the health data to the monitoring tool. The monitoring system can then be configured to expect each application to call the monitoring system periodically to determine health.

### Health Check Publishing in .NET

The Health Check framework provided by .NET includes a publisher you can use for just such scenarios. You register a class that implements the `IHealthCheckPublisher` interface as a singleton. Then, the framework will periodically publish the health of the system using a `PublishAsync` method.

Here is an example:

```csharp
public class LoggingHealthCheckPublisher : IHealthCheckPublisher
{
    private readonly ILogger<LoggingHealthCheckPublisher> _logger;

    public LoggingHealthCheckPublisher(ILogger<LoggingHealthCheckPublisher> logger)
    {
        _logger = logger;
    }
    
    public Task PublishAsync(HealthReport report, CancellationToken cancellationToken)
    {
        switch (report.Status)
        {
            case HealthStatus.Degraded:
                _logger.LogWarning("Health Degraded");
                break;
            case HealthStatus.Unhealthy:
                _logger.LogError("Unhealthy");
                break;
            case HealthStatus.Healthy:
                _logger.LogInformation("Healthy");
                break;
            default:
                throw new ArgumentOutOfRangeException();
        }

        return Task.CompletedTask;
    }
}
```

And here is how it is registered:

```csharp
// Program.cs

builder.Services.AddHealthChecks()
    .AddCheck("self", () => HealthCheckResult.Healthy());

builder.Services.AddSingleton<IHealthCheckPublisher, LoggingHealthCheckPublisher>();
builder.Services.Configure<HealthCheckPublisherOptions>(options =>
{
    options.Delay = TimeSpan.FromSeconds(5);
    options.Period = TimeSpan.FromSeconds(60);
});
```

The `Delay` property defines a one-time delay at startup before publishing the health checks begins. The `Period` property defines the period between calls to the `PublishAsync` method.

Note, it is possible to have more than one registered publisher class.

### Potential Use Cases

In this example, the health check data is written to the log once every 60 seconds. But you could also publish the data to a log monitoring system (like Seq, or Application Insights). Or, you could perform a task. And this could be anything. If the application is healthy, do this thing. Or if the application is not healthy do this other thing. For example, if the application is healthy you could renew a lease the application has on a resource. 

For more details on the health check publishers, check out the docs:

[https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks#health-check-publisher](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks#health-check-publisher)
