---
layout: post
title: 'ASP.NET Code Logging - Lessons Learned'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

The [built-in logging support in ASP.NET Core][aspnet-logging] is wonderful. But there are ways you can shot yourself in the foot if you aren't careful. Here are some lessons I've learned to help make logging better.

<!--more-->

## Why is Logging Important?

Logging allows you to record important or interesting events that occur as you application is running. When developing the software locally, this can be useful in observing the behavior of the application. When running in production, this is usually the only way you can tell what occurred when an issue is reported. Correctly logging events is therefore always useful and doing it properly is very important. 

## Logging with ASP.NET Core

Access to a logger is via the dependency injection framework:

```csharp
public class HomeController : ControllerBase
{
    private readonly ILogger<HomeController> _logger;

    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }
}
```

The generic parameter `T` in `ILogger<T>` represents a logging category associated with each log. This can be used to configure the logging at runtime (the fully-qualified name of the class is the category name).

Creating log entries is done via the ILogger interface:

```csharp
_logger.LogDebug("message");
_logger.LogInformation("message");
_logger.LogWarning("message");
_logger.LogError("message");
```

## Lessons Learned

I have learned a number of lessons over the years with respect to logging. Most of these apply to any framework or technology. However, there are a few that I have learned recently while using ASP.NET Core applications. I figured I better write them down before I forget!

### Use Application Settings to Configure Logging

The team I work with uses [Serilog][serilog] to provide logging services to their applications. It is a very good provider and I have found no reason to use any other (yet). It provides the option to configure the logging services in code, or via the `appsettings.json` file. I cannot stress the importance enough of allowing the logging to be configured with the application settings. It allows the settings to change easily for each environment (local, QA, production, etc.) without having to recompile you code. 

### Log to Console (By Default)

When running the code locally make sure, that by default, the logging entries are sent to the Console. This makes local development efforts a lot easier. It is assumed the non-local environments will have this turned off. But having log entries sent to the Console is incredibly important. And since the settings can be adjusted via the application settings, you can customize and filter messages by log level and category names to suite your needs.

### Use Log Message Templates Correctly

When writing a message using the ASP.NET Core logging, it is important to use the correct format. ASP.NET Core uses message templates to format messages in the log stores. It can be easily overlooked in the documentation:

[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/#log-message-template](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/#log-message-template)

Someone on our team experimented with these messages and verified that Serilog stores both the message string and the message string with the parameters replaced in its store:

```csharp
var id = 123456;
...
_logger.LogInformation("Invalid customer {id}", id);
```
`Invalid customer 123456` is stored in the Message column
`Invalid customer {id}` is stored in the MessageTemplate column

This makes finding entries using a specific template very straightforward.

### Log Startup and Shutdown

It is helpful to know when an application starts and shuts down. You'd be surprised how often this occurs (and when). I have learned that logging the startup and shutdown can be very helpful. Serilog supports this in the `Program.cs` as follows:

```csharp
public static int Main(string[] args)
{
    var configuration = CreateConfigurationBuilder(args).Build();

    Log.Logger = new LoggerConfiguration()
        .ReadFrom.Configuration(configuration)
        .CreateLogger();

    try
    {
        Log.Information("Starting web host");

        CreateWebHostBuilder(args).Build().Run();

        return 0;
    }
    catch (Exception ex)
    {
        Log.Fatal(ex, "Host terminated unexpectedly");
        return 1;
    }
    finally
    {
        Log.Information("Ending web host");
        Log.CloseAndFlush();
    }
}
```

### Allow your Logs to be Traced

When hunting down a bug and all you have to use are your log entries it is important to know where in the code a specific message originated. The best way to do that is to make each message template unique. This allows you to trace through the code and find the exact spot where a template is used. 

Similarly, this can also mean a change to how your code is designed. Don't allow a routine, especially a controller action, to have two or more return paths with the same result. You'll never know which path resulted in a given result which will make you spend a lot of time figuring out what went wrong.

```csharp
public IActionResult UpdateCustomer(int id, [FromBody] model)
{
    if (id == 0)
    {
        return NotFound();
    }
    
    var customer = _service.GetCustomer(id);
    if (customer == null)
    {
        return NotFound();
    }
}
```

Which event caused the 404 response code? Was it an `id` or zero, or was the customer not found? A better approach is this:

```csharp
public IActionResult UpdateCustomer(int id, [FromBody] model)
{
    if (id == 0)
    {
        return NotFound("invalid-id");
    }
    
    var customer = _service.GetCustomer(id);
    if (customer == null)
    {
        return NotFound("unknown-customer");
    }
}
```

### Review and Revise

The last piece of advice is one born of using logging for a number of different technologies, including ASP.NET Core. Regularly review and revise your logging strategy. You may find that things that were important to log at a `Information` level can now be logged at a `Debug` level. Or you may find some events are not worth recording at all. You may find that you turned on `Debug` level logging in the QA environment to diagnose an issue but forgot to return it to only `Warning` level and above. Now you've got megabytes worth of data bogging down your systems resources.

Take the time to regularly check the logs being generated and recorded in each environment.

## Summary

I have learned a lot about logging ASP.NET Core applications. I hope I can apply these lessons on all my projects and continue to learn and grow as a developer. That's where all the fun is :-)

[aspnet-logging]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging
[serilog]: https://serilog.net/
