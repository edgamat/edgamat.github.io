---
layout: post
title: 'Timeouts in ASP.NET Core hosted by IIS'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When developing and maintaining ASP.NET Core applications, it is possible you will encounter timeouts with the deployed application. If you are using IIS to host the application, it is helpful to know how it all strings together.

<!--more-->

I am currently working with an ASP.NET Core REST API endpoint application hosted using IIS. It is developed using .NET Core 2.1, but I don't believe this situation is limited to that version (it probably also occurs with .NET Core 3.1 and the newly released .NET 5.0). 

This week the application responded with timeouts in our test environment. The log files didn't capture sufficient detail to report the timeouts correctly so we started to investigate ways to improve the way the application handles timeouts.

### The Defaults

By default the request timeout for IIS is 2 minutes. 

When a timeout for the request occurs, IIS returns a `502.3 Bad Gateway` response.

### Local Simulation

The first thing I usually want to do when faced with a reported issue is replicate the same error with my local environment. So in the controller action, I put a delay of 3 minutes to force the timeout:

```csharp
await Task.Delay(TimeSpan.FromMinutes(3), token);
```

I ran this code but the timeout never occurred. Instead, the request waited the 3 minutes and then returned the normal response for this action. As it turns out, how you run the code locally influences how the timeouts behave.

The documentation states that the default timeout is 2 minutes, I could not get the request to time-out unless I explicitly set the timeout in the `Web.config` file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <system.webServer>
        <aspNetCore processPath="dotnet"
                    arguments=".\MyWebApplication.dll"
                    requestTimeout="00:02:00"/>
    </system.webServer>
</configuration>
```

However, even this approach only worked if using IIS Express to host the website. Using Kestrel would never trigger the timeout.

From my reading, the hosting model for ASP.NET Core changed with 2.2:

**ASP.NET Core Module**  
[https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/aspnet-core-module?view=aspnetcore-2.2](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/aspnet-core-module?view=aspnetcore-2.2)

**ASP.NET Core In Process Hosting on IIS with ASP.NET Core**  
[https://weblog.west-wind.com/posts/2019/Mar/16/ASPNET-Core-Hosting-on-IIS-with-ASPNET-Core-22](https://weblog.west-wind.com/posts/2019/Mar/16/ASPNET-Core-Hosting-on-IIS-with-ASPNET-Core-22)

It is entirely possible that my experiences may change when the application is upgraded beyond ASP.NET 2.1. If so, I'll try to remember and revisit this post and update it if necessary.

### Handling Operation Canceled Exceptions 

When the timeout occurs, the `CancellationToken` supplied by ASP.NET to the action method is cancelled:

```csharp
public async Task<IActionResult> Get(string id, CancellationToken token)
{
    var data = await _service.GetEntityAsync(id, token); // <= long-running task
    
    return Ok(data);
}
```
I found this article helpful in understanding the process of how these token work when they are cancelled. 

**Using CancellationTokens in ASP.NET Core MVC controllers**  
[https://andrewlock.net/using-cancellationtokens-in-asp-net-core-mvc-controllers/](https://andrewlock.net/using-cancellationtokens-in-asp-net-core-mvc-controllers/)

In my application, when the timeout occurs, the system throws an exception, which looks like a normal `500 Internal Server Error` response. But the web browser is shown a `502.3 Bad Gateway`. 

The article suggests adding an exception filter to respond differently, say a 400 response code:

```csharp
public class OperationCancelledExceptionFilter : ExceptionFilterAttribute
{
    private readonly ILogger _logger;

    public OperationCancelledExceptionFilter(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<OperationCancelledExceptionFilter>();
    }
    public override void OnException(ExceptionContext context)
    {
        if(context.Exception is OperationCanceledException)
        {
            _logger.LogInformation("Request was cancelled");
            context.ExceptionHandled = true;
            context.Result = new StatusCodeResult(400);
        }
    }
}
```

But even including this filter didn't change the response from IIS. It still return the 502.3 response.

### Summary

I don't know if my 'local simulation' of timeouts is 100% accurate, but it feels right. 

You can control the timeout via the `Web.config` file. But the internal ASP.NET Core request processing doesn't get returned to the caller.

IIS will return a `502.3 Bad Gateway` when a timeout occurs. I don't see any way to change that:

[https://github.com/aspnet/AspNetCoreModule/issues/48](https://github.com/aspnet/AspNetCoreModule/issues/48)

