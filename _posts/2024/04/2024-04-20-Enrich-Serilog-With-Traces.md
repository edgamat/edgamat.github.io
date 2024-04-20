---
layout: post
title: 'Enrich Logs with Trace Data Using Serilog'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
I'd like to be able to correlate log entries together using trace data. Diagnostics and Serilog to the rescue?

<!--more-->

For the past few years I have used Serilog in several C# applications to write log entries to a SQL Server database table. This approach was pretty good, but it had a couple of drawbacks as well. It was great that we could query the logs using SQL, but there was no way to correlate log entries together, especially log entries that all related to the same request.

Here is a typical example. An application receives an HTTP request and begins processing it. Log entries are captured at the INFO, WARN or ERROR level as processing is performed. Ideally it would be useful to filter the log entries to only the entries for a single request.

One approach is to assign each request a unique 'correlation id'. Then include that id in the log messages. Something like this:

```csharp
// OrderController.cs
private readonly ILogger<OrderController> _logger;
private readonly OrderService _orderService;

public OrderController(ILogger<OrderController> logger, OrderService service)
{
    _logger = logger;
    _orderService = service;
}

public async Task<IActionResult> Post(OrderModel order)
{
    var correlationId = Guid.NewGuid();

    _logger.LogInformation("{CorrelationId}: Order received", correlationId);

    await _orderService.ProcessOrderAsync(order, correlationId);
...
}

// OrderService.cs
private readonly ILogger<OrderService> _logger;

public OrderService(ILogger<OrderService> logger)
{
    _logger = logger;
}

public Task ProcessOrderAsync(OrderModel order, Guid correlationId)
{
    // Process the order

    _logger.LogInformation("{CorrelationId}: Order processed", correlationId);
}
```

We then could filter the log entries:

```sql
SELECT *
FROM [ApplicationLogs]
WHERE [Message] LIKE '45a340d8-7bd1-4f56-95b3-8ca192ef6094: %'
```

![Before](/assets/img/trace-01.png)

This approach has 2 drawbacks. One, the Message column in the database is not indexed and that makes these queries quite slow. And second, the code must pass the correlation property throughout the application to ensure we had this context data whenever it writes a log entry. 

Rather than adding an index to the Message column (which would consume a lot of resources due to the volume of log entries) we would prefer to have a separate column in the database to store the correlation id values. Simple enough to add a new column to the database tables. But how would we add it to the log entries?

## Diagnostics API

.NET includes a set of classes, called the Diagnostics API, which allows you to capture diagnostic data about your application. To capture the data, you create an `ActivitySource` and an `ActivityListener`. The `ActivitySource`, as the name suggests, creates activities. Each activity represents a unit of work that has a starting point, and a duration. Each activity has a unique identifier. You can attach additional attributes, called tags, to an activity with details of the work being traced. The `ActivityListener` is configured to listen to specific activity sources, and provides an event handler to let you know when an activity starts and when it stops.

An `ActivitySource` is accessed by name, so you can create one and share it throughout your application:

```csharp
using System.Diagnostics;

namespace SampleApp;

internal static class DiagnosticsConfig
{
    public static string SourceName = "SampleApp";

    public static ActivitySource ActivitySource { get; } = new(SourceName);
}
```

Now, in the controller action, we can use it to create a new activity:

```csharp
public async Task<IActionResult> Post(OrderModel order)
{
    using var activity = DiagnosticsConfig.ActivitySource.StartActivity("OrderController.Post");

    _logger.LogInformation("Order received");

    await _service.ProcessOrderAsync(order);

    return Ok();
}
```

Notice, we do not need to create a correlationId, include it in the log messages, and pass it throughout the application. The Diagnostics API takes care of that for us.

Now we create an `ActivityListener`:

```csharp
var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

var listener = new ActivityListener
{
    ShouldListenTo = source => source.Name == DiagnosticsConfig.SourceName,
    Sample = (ref ActivityCreationOptions<ActivityContext> options) => ActivitySamplingResult.AllData,
};

ActivitySource.AddActivityListener(listener);

app.Run();
```

The application is now capturing this activity data. We can see this data, if we add an event handler to the listener:

```csharp
var listener = new ActivityListener
{
    ShouldListenTo = source => source.Name == DiagnosticsConfig.SourceName,
    Sample = (ref ActivityCreationOptions<ActivityContext> options) => ActivitySamplingResult.AllData,
    ActivityStopped = activity =>
    {
        Console.WriteLine($"Activity: {activity.OperationName}, {activity.TraceId}, {activity.SpanId}");
    }
};
```

Then we see this in the console:

```
[05:42:42 INF] Order received
[05:42:42 INF] Order processed
Activity: OrderController.Post, 5cc93ce079f996580ea83732fc0b1a8d, 0c74873f9de4affa
```

The `TraceId` is a 16 byte value that uniquely identifies the correlated set of data captured for a given root activity (this is our new correlation id).
The `SpanId` is an 8 byte value that represents the specific activity. 

You can create activities within activities. When this happens, each activity has a different `SpanId` but all share the same `TraceId`. 

For example, let's add an activity in the `OrderService`:

```csharp
public Task ProcessOrderAsync(OrderModel order)
{
    using var activity = DiagnosticsConfig.ActivitySource.StartActivity("OrderService.ProcessOrderAsync");

    // Process the order

    _logger.LogInformation("Order processed");

    return Task.CompletedTask;
}
```

Now, we see this in the logs:

```
[05:42:42 INF] Order received
[05:42:42 INF] Order processed
Activity: OrderService.ProcessOrderAsync, 5cc93ce079f996580ea83732fc0b1a8d, 87053f5f2f48e9b6
Activity: OrderController.Post, 5cc93ce079f996580ea83732fc0b1a8d, 0c74873f9de4affa
```

As you can see, each of the activities has the same `TraceId` = `5cc93ce079f996580ea83732fc0b1a8d`, which links them all together.

Now we need to store this data in the log entries.

### Serilog Span Enricher

The Serilog community has provided an enricher that solves that for us. Include the `Serilog.Enrichers.Span` NuGet package:

```
dotnet add package Serilog.Enrichers.Span
```

Then, add it in your Serilog configuration (as well as the new columns in the logging table!):

```csharp
var connectionString = builder.Configuration.GetConnectionString("Logging");

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft.AspNetCore", LogEventLevel.Warning)
    .WriteTo.Console()
    .WriteTo.MSSqlServer(
        connectionString,
        new MSSqlServerSinkOptions
        {
            TableName = "ApplicationLogs",
            AutoCreateSqlTable = true,
        }, columnOptions: new ColumnOptions
        {
            AdditionalColumns =
            [
                new SqlColumn { ColumnName = "TraceId", DataType = SqlDbType.VarChar, DataLength = 32 },
                new SqlColumn { ColumnName = "SpanId", DataType = SqlDbType.VarChar, DataLength = 16 },
            ]
        })
    .Enrich.FromLogContext()
    .Enrich.WithSpan()
    .CreateLogger();
```

Or if you are configuring it via the `appsettings.json`:

```json
"Serilog": {
  ...
  "Enrich": [
    "FromLogContext",
    "WithSpan"
  ],
  "WriteTo": [
    {
      "Name": "MSSqlServer",
      "Args": {
        "connectionString": "Logging",
        "tableName": "ApplicationLogs",
        "autoCreateSqlTable": true,
        "columnOptionsSection": {
          "additionalColumns": [
            {
              "ColumnName": "TraceId",
              "DataType": "varchar",
              "DataLength": 32,
              "AllowNull": true
            },
            {
              "ColumnName": "SpanId",
              "DataType": "varchar",
              "DataLength": 16,
              "AllowNull": true
            }
          ]
        }
      }
    }
  ]
},
```

Now, the `TraceId` and `SpanId` values are stored in the database, and we can filter our queries with them:

```sql
SELECT *
FROM [ApplicationLogs]
WHERE  TraceId = 'fb445c3cd1cbc73e7a7fb892a3ba3d85'
```

![After](/assets/img/trace-02.png)

### Summary

I wanted a way to correlate log entries together within an existing framework of using Serilog to capture logs in a SQL Server database table. While I demonstrated a way to accomplish this goal, it is only the tip of the iceberg when it comes to tracing activity in your application. If you want to grow beyond this and start answering more questions about your applications, I'd encourage you to look into distributed tracing using OpenTelemetry, which I wrote about last month:

[Distributed Tracing in .NET - Part 0 - The Sample Application]({% post_url /2024/03/2024-03-20-Distributed-Tracing-Part-0 %})  
[Distributed Tracing in .NET - Part 1 - Setup Open Telemetry]({% post_url /2024/03/2024-03-20-Distributed-Tracing-Part-1 %})  
[Distributed Tracing in .NET - Part 2 - Exception Handling]({% post_url /2024/03/2024-03-22-Distributed-Tracing-Part-2 %})  
[Distributed Tracing in .NET - Part 3 - Adding Custom Dimensions]({% post_url /2024/03/2024-03-31-Distributed-Tracing-Part-3 %})  
