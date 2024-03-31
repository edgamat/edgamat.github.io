---
layout: post
title: 'Distributed Tracing in .NET - Part 3 - Adding Custom Dimensions'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In the previous post we described how to enable distributed tracing in .NET applications using OpenTelemetry. In this post I want to enrich the traces with custom dimensions.

<!--more-->

### Dimensions in Telemetry Data

Dimensionality in telemetry data refers to the number of keys included in each structured event. The OpenTelemetry .NET SDK libraries automatically adds several. Here is an example trace event captured in Seq for a given span:

![Seq Trace Dimensions](/assets/img/otel-04.png)

Each of the keys represents a dimension. You can use Seq to search/filter on any combination of these keys and values. One of the benefits of OpenTelemetry and observability is that you can add as many dimensions (key/value pairs) as you like. 

### Adding Custom Dimensions

You can add custom 'tags' to the telemetry data using the ambient `Activity` instance provided by the built-in diagnostics of .NET:

```csharp
Activity.Current?.SetTag("my-key", "my-value");
```

Each one of these tags is attached to the current _span_, and will be visible in Seq. So these values do not travel from one span to the next. If you would like this behavior, you must do it manually.

So... what should you add?

### Good Candidates for Custom Dimensions

Here are some good candidates to instrument as tags:

- user identifiers
- deployment environment (dev/test/prod)
- customer identifiers
- business entity identifiers (e.g. customer id, product id)

You should add as many tags as you see fit. There is no penalty for adding too many dimensions. Think about what data would be useful when diagnosing an issue or measuring something. Add those as tag values. 

### Tag Naming Conventions

In OpenTelemetry, tags are called "Attributes". Here is the guidance they suggest for naming attributes:

[https://github.com/open-telemetry/semantic-conventions/blob/main/docs/general/attribute-naming.md](https://github.com/open-telemetry/semantic-conventions/blob/main/docs/general/attribute-naming.md)

Here is a summary of the conventions:

- Every name MUST be a valid Unicode sequence.
- Names SHOULD be lowercase.
- Use namespaces to avoid name clashes. Namespace components are separated by a dot (e.g. `service.version`)
- each component that has multiple words should have them separated by underscore (e.g. `status_code` in `http.response.status_code`)


Here are some reference documents that describe some other conventions to consider:

General:  
[https://github.com/open-telemetry/semantic-conventions/blob/main/docs/general/attributes.md](https://github.com/open-telemetry/semantic-conventions/blob/main/docs/general/attributes.md)

Traces:  
[https://github.com/open-telemetry/semantic-conventions/blob/main/docs/general/trace.md](https://github.com/open-telemetry/semantic-conventions/blob/main/docs/general/trace.md)

Resources:  
[https://github.com/open-telemetry/semantic-conventions/blob/main/docs/resource/README.md](https://github.com/open-telemetry/semantic-conventions/blob/main/docs/resource/README.md)

To ensure consistency, it is recommended that tag names be stored as named constants:

```csharp
public static class DiagnosticNames
{
    public const string PersonFirstname = "person.firstname";
    public const string PersonSurname = "person.surname";
}
```
### Adding Events

Events can be added to spans to record when something occurred within the lifetime of the span. For example, you can record an event when you evaluate a feature flag. This allows you to know what the feature flag value was at a given point in time. 

Here is an example:

```csharp
var eventTags = new Dictionary<string, object?>
{
    { "invoice.id", invoiceId },
    { "invoice.customer_id", invoice.CustomerId}
};
Activity.Current?.AddEvent(new ActivityEvent("invoice-sent", tags: new(eventTags)));
```

When using Seq, events appear as log entries, linked to the span.

![Seq Trace Event](/assets/img/otel-06.png)

### Summary

Adding custom dimensions means enriching your spans with additional detail that can be critical in answering questions about the state of your application. Find data values that are important in identifying the current operation and add those to the span as tags.
