---
layout: post
title: 'Using ASP.NET Core Health Checks'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

After using a custom health check solution I was curious to see what the new HealthChecks library from Microsoft can offer.

<!--more-->

### Monitoring Your Application

I worked on a suite of ASP.NET Core applications that used an external monitoring solution to report the health of the applications. Each application exposed an HTTP Endpoint that reported the status of the application. A response status code of 200 would indicate a healthy application, while a response status code of 503 would indicate an unhealthy one. The body of the response would contain a JSON payload with some high-level information for the application and each of the health checks. This meant you could potentially see which health checks were unhealthy.

After this custom solution was in place, [Microsoft released its own health check solution][docs]. If I was to start a similar project today, I would want to see if this new offering would provide the same level of monitoring functionality.

### Adding Health Checks

*NOTE*: All these examples are using ASP.NET applications with .NET Core 5.0

You need to register the health check services in order to add the health check endpoint. This is done via the ConfigureService method in the `Startup` class:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks();
}
```

Once registered, you need to add the routing information to define what URL the health check endpoint uses: 


```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHealthChecks("/health");
    });
}
```

Pointing a web browser to your running application should give you a healthy response:

![sample showing response from the endpoint](/assets/img/health-check-endpoint.png)

### Adding Health Probes

The Health Check library provides an interface `IHealthCheck` to create custom health check probes:

```csharp
public interface IHealthCheck
{
    Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default);
}
```

The `CheckHealthAsync` method returns a `HealthCheckResult` that indicates the health status:

```csharp
public enum HealthStatus
{
    Unhealthy = 0,

    Degraded = 1,

    Healthy = 2
}
```

Here is a health probe that verifies that the log path exists:

```csharp
public class LogPathHealthCheck : IHealthCheck
{
    private readonly IConfiguration _configuration;

    public SimpleHealthCheck(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken)
    {
        var logPath = _configuration["LogPath"];

        if (Directory.Exists(logPath))
        {
            return Task.FromResult(HealthCheckResult.Healthy());
        }

        return Task.FromResult(HealthCheckResult.Unhealthy());
    }
}
```

You register it like this:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddCheck<LogPathHealthCheck>();
}
```

### Customizing the Response

The custom system we used included the ability to provide the status of each registered health check in the response. This made it possible to easily tell which probe was unhealthy. The Microsoft offering provides this ability as well. You can write a custom method to write out the response, rather than the default 'Healthy', 'Degraded' or 'Unhealthy' plain-text response.

You can set which method to use via  the `HealthCheckOptions`:

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHealthChecks("/health", new HealthCheckOptions()
        {
            ResponseWriter = JsonResponseWriter.WriteResponse
        });
    });
```

```csharp
public static class JsonResponseWriter
{
    public static Task WriteResponse(HttpContext context, HealthReport result)
    {
        context.Response.ContentType = "application/json; charset=utf-8";

        var options = new JsonWriterOptions { Indented = true };

        using var stream = new MemoryStream();
        using (var writer = new Utf8JsonWriter(stream, options))
        {
            writer.WriteStartObject();
            writer.WriteStartObject("results");

            foreach (var entry in result.Entries)
            {
                writer.WriteStartObject(entry.Key);
                writer.WriteString("status", entry.Value.Status.ToString());
                writer.WriteEndObject();
            }
            
            writer.WriteEndObject();
            writer.WriteEndObject();
        }

        var json = Encoding.UTF8.GetString(stream.ToArray());
        return context.Response.WriteAsync(json);
    }
}
```

### Summary

The new offering from Microsoft provides a functional replacement for the custom solution we have been using. I hope to provide a couple more examples in future blog posts on how it can be customized further.

[docs]: https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-5.0
