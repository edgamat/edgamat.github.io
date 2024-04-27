---
layout: post
title: 'Configuring Serilog to use OpenTelemetry'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
How difficult is it to use OpenTelemetry with Serilog?

<!--more-->

I've been spending a lot of time recently exploring distributed tracing and OpenTelemetry. I've worked in environments that have multiple services working together to process business operations. We didn't have distributed tracing or centralized logging and it made supporting these applications difficult. We relied too heavily on individuals that 'just knew' where to look for errors and problems. Moving forward I want to do better. I want everyone to have access to the telemetry data and for anyone who is curious to be able to figure out why an application is behaving the way it does.

Many .NET applications use a logging framework, like Log4Net or Serilog. In the .NET applications that I helped develop, we have used Serilog for several years and are very pleased with its capabilities. But we have a couple of things we would like to improve. 

First, we want to store all the logs in a centralized logging service. Currently each application stores its own logs separately, using the Serilog SQL Server sink. This makes it challenging to diagnose issues. Not everyone has access to these SQL Server databases. And the only way to search the logs is using SQL statements, which doesn't make it easy to filter by custom attributes and to visualize the data.

Second, we do not capture any trace data (or metrics). Distributed tracing data is the real game changer for diagnosing issues for a collection of deployed services. You can follow the journey of a request as it flows through the system and understand the context when investigating problems.

One option would be to replace Serilog with a different telemetry API. There are many vendors that provide such APIs for their products. But in this context, I want to explore a simpler approach. I want to continue to use Serilog, but reconfigure it to use a centralized logging service. There are a log of choices for such a service: Azure Application Insights, Seq, Zipkin, Jaeger, Honeycomb, Datadog, New Relic, Prometheus, AWS CloudWatch, Elasticsearch, and so on. Some of these products have existing sinks for Serilog, developed by the Serilog community. But what I'd like to do is explore using OpenTelemetry.

OpenTelemetry is a set of protocols and a toolkit for using a vendor-agnostic way to export telemetry data from applications. That means that I can use a single Serilog Sink and send the telemetry data to a wide range of backends (logging services) that support the ingestion of OpenTelemetry data.

Here is the typical Serilog setup our applications use:

```csharp
var connectionString = builder.Configuration.GetConnectionString("Logging");

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.MSSqlServer(
        connectionString,
        new MSSqlServerSinkOptions
        {
            TableName = "ApplicationLogs",
            AutoCreateSqlTable = true,
        })
    .Enrich.FromLogContext()
    .CreateLogger();
```

One of the logging services that supports OpenTelemetry is Seq ([https://datalust.co/seq](https://datalust.co/seq)). I have an instance of this service running locally. I'd like to send my telemetry data there.

To begin, let's install the OpenTelemetry sink:

```
dotnet add package Serilog.Sinks.OpenTelemetry
```

Then let's reconfigure the application to use the OpenTelemetry sink:

```json
// appsetting.json
{
  "OtelEndpoint": "http://localhost:5341/ingest/otlp/v1/logs"
}
```

```csharp
var otelEndpoint = builder.Configuration.GetValue<string>("OtelEndpoint") ?? "";

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .WriteTo.OpenTelemetry(options =>
    {
        options.Endpoint = otelEndpoint;
        options.Protocol = Serilog.Sinks.OpenTelemetry.OtlpProtocol.HttpProtobuf;
    })
    .Enrich.FromLogContext()
    .CreateLogger();
```

The options (the Endpoint and Protocol) were something I figured out looking at the Seq documentation. Each vendor will have its own way of ingesting OpenTelemetry data, so it is best to consult those docs when choosing a backend logging service.

And well, it worked! 

It is also possible to configure resource (i.e. application) attributes to include with every log entry. This additional detail can help a lot when diagnosing issues:

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

I am glad to have discovered that configuring Serilog to export log entries using OpenTelemetry is straightforward. Next, we need to see what level of support Serilog has for exporting trace data!
