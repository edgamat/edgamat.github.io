---
layout: post
title: 'Choosing the Correct LogLevel in ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

ASP.NET Core includes a very powerful and easy to use logging framework. It provides you with a lot of options, including the level you want the log entry to be made. It can be tricky to choose the correct level so let's explore it a bit.

**Assumption**  I assume you are using the logging API to store logging messages to a persistent store, like a file, or a database. 

<!--more-->

### Logging in ASP.NET

There is a great document provided by Microsoft that describes it's logging API:

**Logging in .NET Core and ASP.NET Core**  
[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging)

I suggest you give it a read if you aren't familiar with the logging API. In a nutshell, you use the Dependency Injection (DI) container to register the logging services and then inject loggers into your application code when needed. 

### Configuring the Proper Logger

When you inject a logger into your object, you choose the logging category by the type of logger you use:

```csharp
namespace MyProject.Api
{
    public class MyController
    {
        private readonly ILogger<MyController> _logger;
        
        public MyController(ILogger<MyController> logger)
        {
            _logger = logger;
        }
    }
}
```

The logging category is a string containing the fully-qualified class name of `MyController`, "MyProject.Api.MyController". You can use this string in the `appsettings.json` to control the logs being created:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
      "MyProject.Api.MyController": "Warning"
    }
  }
}
```

This will log all messages `Information` level or higher except for messages in the "MyProject.Api.MyController" category. For this category, only messages `Warning` level or higher will be logged.

### The Logging Levels

Here are the 'standard' logging levels used by the Logging API, and how Microsoft intended them to be used:

 LogLevel | Value | Method | Description
----------|-------|--------|------------
Trace | 0 | LogTrace | Contain the most detailed messages. These messages may contain sensitive app data. These messages are disabled by default and should not be enabled in production.
Debug | 1 | LogDebug | For debugging and development. Use with caution in production due to the high volume.
Information | 2 | LogInformation | Tracks the general flow of the app. May have long-term value.
Warning | 3 | LogWarning | For abnormal or unexpected events. Typically includes errors or conditions that don't cause the app to fail.
Error | 4 | LogError | For errors and exceptions that cannot be handled. These messages indicate a failure in the current operation or request, not an app-wide failure.
Critical | 5 | LogCritical | For failures that require immediate attention. Examples: data loss scenarios, out of disk space.
None | 6 |  | Specifies that a logging category should not write any messages.

## Choosing What to Log in an Environment

As demonstrated above, ASP.NET applications can choose what level to create logs by changing levels in the `appsettings.json` file. This means the application can be configured differently for each target environment. What you choose to log on your local workstation during development can be much different than what is logged in a Production environment. Here is an example plan you may choose to use:

LogLevel | Development | Staging | Production
---------|-------------|---------|-----------
Trace | X | - | -
Debug | X | - | -
Information | X | ? | ?
Warning | X | X | X
Error | X | X | X
Critical | X | X | X

For Warning, Error and Critical Log Levels, You should probably log all of these, all the time. They should be infrequent and provide insights into issues that need attention.

It is not a good idea to configure your logging to produce messages at the `Debug` and `Trace` level except when developing locally. It should be well understood by all developers that they can use these to their advantage when developing locally, without concern of them being persisted to storage in any of the deployed environments. And the operations staff should also assume that there is never a need to enable logging at these levels (unless asked for explicitly, in order to diagnose an issue in a deployed environment).

That leaves the `Information` level. It is probably the hardest thing to get correct. When you are developing the software, you may choose to add a log entry at the `Information` level because it helps give you insight into the normal flow of processing requests:

- Log the identifiers of a given request, so you can know the data that was being processed at a particular time. 
- Log when the flow enter or exits a given state ('Starting' or 'Stopping' a task)
- Log when calling an external API or service

But this can lead (rather quickly) to so many log entries being recorded that the storage of these entries becomes unsustainable. 

### So what should you do?

It has been my experience that most entries recorded at the `Information` level only help diagnose issues that may arise. In a way, they are really `Debug` level messages. So it may be wise to change the log level to `Debug` and not record them in Staging/Production environments.

However, you can eliminate them all. I have found the best thing is to use `Information` log level entries only where knowing something in the Staging/Production environments is useful for diagnosing issues that occur. 

If the amount of log entries at the `Information` level becomes overwhelming, use the overrides in the `appsettings.json` to filter out the entries (or 'noise') by category.

Using my example above, you can set the default log level to `Information` for all categories. Then, for a given category, you can set the warning level higher so that the `Information` level messages are no longer logged:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "MyProject.Api.MyController": "Warning"
    }
  }
}
```

This provides the necessary flexibility to strike a balance between Information log entries that provide value in Production and those that are noise.

### Analyze and Revise

I recommend you analyze the log entries in each environment periodically to determine if the log entries being record are still useful in the given environment. You may find that some messages you once relied on are no longer as important. You may also find that the settings may be configured incorrectly and need to be put back to their proper logging levels. 


### Summary

Recording log entries is very useful in software development and the Logging API within ASP.NET Core applications is very flexible. Choose the level for each entry that makes sense. Then configure the logging API to only record entries at the proper level, depending on the environment. If you need to only record some messages within a given log level, include/exclude the entries based on their category names.

And don't forget to periodically look at the logs to see if any of these settings need adjustment.
