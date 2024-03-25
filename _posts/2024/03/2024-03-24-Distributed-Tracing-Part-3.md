---
layout: post
title: 'Distributed Tracing in .NET - Part 3 - Adding Custom Dimensions'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
published: false
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

### Tag Naming Conventions

