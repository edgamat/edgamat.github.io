---
layout: post
title: 'Distributed Tracing in .NET - Part 2 - Exception Handling'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In the previous post we described how to enable distributed tracing in .NET applications using OpenTelemetry. In this post I want to enrich the automatic instrumentation with application specific data when exceptions occur.

<!--more-->

### Emit Exceptions

Distributed tracing is a diagnostic technique that helps engineers find/investigate failures and performance issues within applications, especially those distributed across multiple machines or processes. When an exception if thrown processing a request in an ASP.NET Web API application, the framework automatically logs an exception and returns a 500 Internal Server Error status code in the response. We want to include these exception details in the data exported by OpenTelemetry.

Currently, you have to set an environment variable to enable this behavior (it should be corrected in .NET 9). For now, set the following environment variable:

`OTEL_DOTNET_EXPERIMENTAL_OTLP_EMIT_EXCEPTION_LOG_ATTRIBUTES="true"`  
(Its value must be a string, not a boolean value)

See this page for further details: [https://github.com/datalust/seq-tickets/discussions/2119](https://github.com/datalust/seq-tickets/discussions/2119)

### ProblemDetails Exception Responses

We should be using the standardized ProblemDetails response format ([https://www.rfc-editor.org/rfc/rfc7807.html](https://www.rfc-editor.org/rfc/rfc7807.html)) when returning exception details to the caller of our Web API endpoints.

To enable this, register the services required for the creation of ProgramDetails:

```csharp
builder.Services.AddProblemDetails();
```

And enable the default error handler:

```csharp
var app = builder.Build();

app.UseExceptionHandler();
```

Here is what the API response looks like:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.6.1",
  "title": "An error occurred while processing your request.",
  "status": 500
}
```

### Including Tracing Details

To find the log and trace data in your log analysis tool, like Seq, it would be useful to know the trace identifier for the failed request. You can customize the ProblemDetails response to include this value:

```csharp
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = (context) =>
    {
        var traceId = Activity.Current?.Id ?? context.HttpContext.TraceIdentifier;
        if (traceId != null)
        {
            context.ProblemDetails.Extensions["traceId"] = traceId;
        }
    };
});
```

Now the response looks like this:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.6.1",
  "title": "An error occurred while processing your request.",
  "status": 500,
  "traceId": "00-f078be0e6a4fdb30011bf0b87763f646-006b4d238e5c47d6-01"
}
```

The traceId includes 4 components:

- '00'
- 'f078be0e6a4fdb30011bf0b87763f646' => this is the TraceId exported by OpenTelemetry
- '006b4d238e5c47d6' => this is the SpanId
- '01'

The TraceId exported by OpenTelemetry is what you can search for in Seq.

`@TraceId = "f078be0e6a4fdb30011bf0b87763f646"`

### Summary

Exporting exception details is important for observability of your application's behavior. When evaluating any distributed tracing solution, ensure this is the case! And wherever possible, provide the trace identifier in order to more quickly find the trace data for exceptions in your log analysis tool.
