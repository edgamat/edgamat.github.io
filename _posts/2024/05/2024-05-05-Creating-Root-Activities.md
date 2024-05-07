---
layout: post
title: 'Creating Root Activities in C#'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
I recently had a need to create separate traces for each item in a batch. This introduced me to 'root' activities.

<!--more-->

### The Problem

One of the background tasks in a C# application I support is responsible for processing batches of items. The batch is retrieved from the database and then each item is processed sequentially. I wanted to understand how it behaves in production so I added activities using the .NET Diagnostics API:

```csharp
private static readonly ActivitySource ActivitySource = new("tracing-root-traces");

private async Task ProcessBatchAsync(CancellationToken stoppingToken)
{
    using var batchActivity = ActivitySource.StartActivity("ProcessBatch");

    var batch = await GetBatchAsync(stoppingToken);

    foreach (var item in batch.Items)
    {
        using var itemActivity = ActivitySource.StartActivity("ProcessItem");

        await ProcessItemAsync(item, stoppingToken);
    }
}
```

I added an `ActivityListener` that wrote to the console some details of each activity:

```csharp
logger.LogInformation("Activity stopped: {OperationName} {ParentId} {TraceId} {SpanId}",
    activity.OperationName, activity.Parent?.TraceId, activity.TraceId, activity.SpanId);
```

Here's what it produced:

```
Activity stopped: ProcessItem 79b3b07085973d9b2e7933ab12b5c07b 79b3b07085973d9b2e7933ab12b5c07b 0cb209819cd2b59d
Activity stopped: ProcessItem 79b3b07085973d9b2e7933ab12b5c07b 79b3b07085973d9b2e7933ab12b5c07b b0d7664bda92411a
Activity stopped: ProcessItem 79b3b07085973d9b2e7933ab12b5c07b 79b3b07085973d9b2e7933ab12b5c07b c08cf72264646f2a
Activity stopped: ProcessBatch (null) 79b3b07085973d9b2e7933ab12b5c07b cbce20d22ee75b08
```

As you can see, each "ProcessItem" activity shares the same `TraceId` as the parent "ProcessBatch" activity. 

It's helpful, but what I really was hoping for was a separate trace for each item being processed. I wanted to know how much time it took to process the entire batch. But for each item being processed, there are several child spans that I would like to link together in order to view all logs for a given item. For that I needed to use root activities.

### Root Activities

A root activity is one whose parent is either `null` or itself. It isn't a child of a parent activity, and therefore has its own unique `TraceId`.

When starting a new activity, the Diagnostics API determines if there is an existing activity context to use as a parent. What we need to do is explicitly tell the API to ignore the current context and use a custom one that we provide.

To create a context you need to create a `TraceId` and a `SpanId`. There are functions that provide these for you:

```csharp
var traceId = ActivityTraceId.CreateRandom();
var spanId = ActivitySpanId.CreateRandom();
```

One additional property we require is a TraceFlag. This is part of the W3C tracing standard ([https://www.w3.org/TR/trace-context/](https://www.w3.org/TR/trace-context/)). It indicates if the activity is recorded or not. We can use the flag that is set for the current context: `Activity.Current.ActivityTraceFlags`. 


With these 3 values we can construct a new context for our activity:

```csharp
var rootContext = new ActivityContext(
    ActivityTraceId.CreateRandom(),
    ActivitySpanId.CreateRandom(),
    Activity.Current.ActivityTraceFlags);
```

To make it easier to integrate into the code I created a simple extension method:

```csharp
using System.Diagnostics;

internal static class ActivitySourceExtensions
{
    public static Activity? StartRootActivity(this ActivitySource source, string name)
    {
        if (Activity.Current is null)
        {
            return null;
        }

        var rootContext = new ActivityContext(
            ActivityTraceId.CreateRandom(),
            ActivitySpanId.CreateRandom(),
            Activity.Current.ActivityTraceFlags);

        return source.StartActivity(name, ActivityKind.Internal, rootContext);
    }
}
```

We can swap out the "StartActivity" call with our new method:

```csharp
        using var itemActivity = ActivitySource.StartActivity("ProcessItem");
```

And here are the results:

```
Activity stopped: ProcessItem (null) 49b8db00da0559738fe6a769f5690d55 321c128842c0beb0
Activity stopped: ProcessItem (null) 948002a01a0c77016e6fe82a5eb9424e 5deb34539bec28e2
Activity stopped: ProcessItem (null) 639263d0776f1209f5db72d132e08873 fd428f6eaf4b6ebc
Activity stopped: ProcessBatch (null) 1b410a4fa883695cd18f0734c542fa55 b52c81d6d0e3d7db
```

As you can see each activity has its own unique TraceId, as we desired. All the child activities under each "ProcessItem" activity will all share the same `TraceId` and can be filtered to view all the logs for a given item. And for the batch I can still record how long it took to process. 

### Summary

When the situation arises to break the chain of activities being recorded, create root activities, which don't have a parent. The Diagnostics API makes this possible by providing the ability to create an explicit context for your activity with unique trace and span ids.
