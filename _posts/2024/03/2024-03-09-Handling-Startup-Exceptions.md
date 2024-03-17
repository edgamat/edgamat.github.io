---
layout: post
title: 'Handling ASP.NET Startup Exceptions'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

This week we mis-configured a deployment of our ASP.NET application. It leaves it in an awful state with very little details. I wanted to see if I could do better.

<!--more-->

### The Problem

I was refactoring how our ASP.NET 7.0 application was configured. I ran our deployment pipeline and it deployed successfully into the DEV environment (as an IIS website). But when I went to the home page I was greeted with sadness:

![Startup Exception](/assets/img/startup-error.png)

After checking the Event Logs on the server, we determined that it was a runtime error that occurred before the application was running:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();
...
// Something blew up here!
...
var app = builder.Build();
...
app.Run();
```

### The Solution

To handle these situations better in the future, the first thing we did is add an exception handler that logs the exception details for us:

```csharp
try
{
    var builder = WebApplication.CreateBuilder(args);
    ...
    var app = builder.Build();
    ...
    app.Run();
} 
catch (Exception ex)
{
    Logger.LogError(ex, "Unhandled Startup Exception");
    throw;
}
```
This allowed us to view the error in more detail, using our normal logging solution (we don't have to remote into the server to look at the Event Logs). Great, so now we know more about the error. But it still leaves the website in a state that is less than ideal. What I'd like to do is show the user a more appropriate web page, compared to the 500.30 message shown above.

In the exception handler, I decided to try spinning up a new web application:

```csharp
catch (Exception ex)
{
    Logger.LogError(ex, "Unhandled Startup Exception");
    
    var startupExceptionApp = new WebApplication.CreateBuilder(args);
    
    startupExceptionApp.Run();
}
```

Instead of getting the 500.30 error, I get a 404 NOT FOUND response. I created a middleware class to handle all requests:

```csharp
public class StartupExceptionHandlerMiddleware
{
    private readonly RequestDelegate _next;

    public StartupExceptionHandlerMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext httpContext)
    {
        var path = httpContext.Request.Path;
        if (path != null)
        {
            // Check if the path is not '/health'
            if (!path.Equals("/health", StringComparison.OrdinalIgnoreCase))
            {
                // Custom processing for all other paths
                httpContext.Response.StatusCode = (int)HttpStatusCode.ServiceUnavailable;
                httpContext.Response.ContentType = "text/html";
                using var file = File.OpenRead("StartupException.html");
                await file.CopyToAsync(httpContext.Response.Body);

                return; // Important to prevent calling next middleware for these paths
            }
        }

        // Proceed with the next middleware (important for the '/health' endpoint to work)
        await _next(httpContext);
    }
}
```

The `StartupException.html` file defines a more appropriate web page for users to view when a startup exception has occurred.

Then I use the middleware in the startup exception web application:

```csharp
    var startupExceptionApp = new WebApplication.CreateBuilder(args);

    startupExceptionApp.UseMiddleware<StartupExceptionHandlerMiddleware>();
    
    startupExceptionApp.Run();
```

This gave me the experience I was hoping for. If the 'real' web application fails to run, a bare-minimum web application takes its place and  presents users with a reasonable experience while we address the error and get it fixed. 

### Health Checks

I also wanted to expose a health check endpoint when an error occurred. To do that I created a handler that encapsulated all the steps to handle a startup exception:

```csharp
public static class StartupExceptionHandler
{
    public static void Handle(Exception ex, string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        builder.Services.AddHealthChecks()
            .AddCheck("StartupFailure", () => HealthCheckResult.Unhealthy(ex.Message, ex));

        var app = builder.Build();

        app.MapHealthChecks("/health", new HealthCheckOptions()
        {
            ResponseWriter = JsonResponseWriter.WriteResponse
        });

        app.UseMiddleware<StartupExceptionHandlerMiddleware>();

        app.Run();
    }
}
```

Then it replaced the logic in the `catch` block:

```csharp
try
{
    var builder = WebApplication.CreateBuilder(args);
    ...
    var app = builder.Build();
    ...
    app.Run();
} 
catch (Exception ex)
{
    Logger.LogError(ex, "Unhandled Startup Exception");
    StartupExceptionHandler.Handle(ex, args);
}
```

### Limitation

I'd like to qualify this solution with an obvious limitation. It is possible that the runtime exception that the 'real' application encountered may also occur with the bare-minimum version I added. In such cases, this additional work was all for not. In my experience this is rare, but not unheard of. So don't be surprised if this doesn't work 100% of the time. 
