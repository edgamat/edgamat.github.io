---
layout: post
title: 'Exporting Trace Data with SerilogTracing'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
In my last post I showed how to export logs from a .NET application using Serilog and OpenTelemetry. Now let's export traces.

<!--more-->

Serilog has a minimal tracing framework you can use in concert with its existing logging framework. The goal is to provide an easy mechanism to export traces without having to overhaul your entire logging infrastructure.

### Install SerilogTracing

The library is called SerilogTracing and is installed via NuGet packages, just like Serilog. To start, I added the package to my existing application:

```
dotnet add package SerilogTracing
```

At this time, only three sinks are capable of receiving the traces using SerilogTracing: the Seq, Zipkin, and OpenTelemetry sinks. 

Here is the logging configuration I am starting with, using the OpenTelemetry sink:

```csharp
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.OpenTelemetry(options =>
    {
        options.Endpoint = otelEndpoint;
        options.Protocol = Serilog.Sinks.OpenTelemetry.OtlpProtocol.HttpProtobuf;
        options.ResourceAttributes = new Dictionary<string, object>
        {
            ["service.name"] = "MyAppName",
            ["service.version"] = "1.0.0",
            ["deployment.environment"] = "Local"
        };
    })
    .Enrich.FromLogContext()
    .CreateLogger();
```

To start exporting traces, we need to remove the Serilog Sink for OpenTelemetry and replace it with a SerilogTracing forked version:

```
dotnet remove package Serilog.Sinks.OpenTelemetry
dotnet add package SerilogTracing.Sinks.OpenTelemetry
```

Our configuration changes slightly, we now have 2 separate endpoints, one for logs and the other for traces:

```csharp
var otelEndpointLogs = "http://localhost:5341/ingest/otlp/v1/logs";
var otelEndpointTraces = "http://localhost:5341/ingest/otlp/v1/traces";

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.OpenTelemetry(options =>
    {
        options.TracesEndpoint = otelEndpointTraces;
        options.LogsEndpoint = otelEndpointLogs;
        options.Protocol = OtlpProtocol.HttpProtobuf;
        options.ResourceAttributes = new Dictionary<string, object>
        {
            ["service.name"] = "MyAppName",
            ["service.version"] = "1.0.0",
            ["deployment.environment"] = "Local"
        };
    })
    .Enrich.FromLogContext()
    .CreateLogger();
```

In the `Program.cs` file, we have one last addition. We need to add a listener to enable sending the trace data to the configured Serilog logger:

```csharp
using var _ = new ActivityListenerConfiguration().TraceToSharedLogger();
```

Once this is in place we need to start adding activities to our code.

### Using SerilogTracing

Here is an example of starting an activity. As soon as the object is disposed, the activity data is written to the logger:

```csharp
using var activity = Serilog.Log.Logger.StartActivity("Do Work");
```

One thing to note, while this mechanism to add activities (spans) builds on the .NET Diagnostics API, it is not 100% compatible. You will need to use the Serilog.Log.Logger to start activities rather than the `ActivitySource`:

```csharp
// .NET Diagnostics API
internal static class DiagnosticsConfig
{
    public static string SourceName = "SampleApp";

    public static ActivitySource ActivitySource { get; } = new(SourceName);
}

using var activity = DiagnosticsConfig.ActivitySource.StartActivity("Do Work");

// SerilogTracing API
internal static class DiagnosticsConfig
{
    public static string SourceName = "SerilogTracing"; 
    
    public static Serilog.ILogger ActivitySource => Log.Logger;
}

using var loggerActivity = DiagnosticsConfig.ActivitySource.StartActivity("Do Work");
```

You will notice that `SourceName` is "SerilogTracing". This is the source name that SerilogTracing uses, and cannot be modified/overridden. SerilogTracing uses a wrapper class around the `Activity` class called `LoggerActivity`. This means that the API it exposes to enrich the activities is limited. Adding events and links to an activity are not supported. If you add them to the activity directly they are not exported. However it is possible to add properties (tags) to activities. This includes the ability to add tags to the ambient activity:

```csharp
    Activity.Current?.AddTag("order.id", order.OrderId);
```

### Observations/Recommendations

At this time (April 2024), SerilogTracing provides a relatively straightforward path to export traces from a .NET application. If you are already using Serilog, this can be a great way to get started with tracing, as it doesn't require a significant change in how you configure Serilog. However, it is really only possible if you are using one of the supported sinks, namely Seq, Zipkin or OpenTelemetry. 

If I was starting a new .NET application (.NET 6 or higher), I would recommend using OpenTelemetry directly. It is simpler to configure and provides more options for capturing custom properties (tags), events and links to the trace data.
