---
layout: post
title: 'Logging Diagnostic Activities in .NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

The Diagnostics API in .NET provides a great way to record activities in your code using the new `ActivitySource` and `Activity` objects. But how do you listen to these activities (and log them)?

<!--more-->

Let's start with a simple scenario, you want to record an activity within a background service. In this case, the service calls a `DoWorkAsync` method, which is the activity we want to record.

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        await DoWorkAsync(stoppingToken);
        await Task.Delay(1000, stoppingToken);
    }
}
```

We can add an activity to record a span:

```csharp
public static readonly ActivitySource Source = new("Tracing.Sample", "1.0.0");

private async Task DoWorkAsync(CancellationToken stoppingToken)
{
    using var activity = Source.StartActivity(DiagnosticsNames.DoWork);

    var workId = Guid.NewGuid().ToString();
    activity?.SetTag("work-id", workId);
...
...
```

The first thing you will notice when stepping through this code using a debugger is the activity object is `null`. This is why it is necessary to use the null-conditional operator to call the `SetTags` method. In order for the `activity` object to not be `null`, you need to configure an activity listener.

### Activity Listeners

An `ActivityListener` is type within the Diagnostics API you can use to listen to the activities being recorded. It gives you a lot of flexibility to control if you want to ignore certain activities or if you want to only record a sample of the activities.

Here is a simple listener that records all activities to the console:

```csharp
var consoleListener = new ActivityListener();
consoleListener.ShouldListenTo = _ => true;
consoleListener.Sample = (ref ActivityCreationOptions<ActivityContext> _) => ActivitySamplingResult.AllData;
consoleListener.ActivityStopped = LogActivityStopped;

public static void LogActivityStopped(Activity activity)
{
    Console.WriteLine($"Trace:{activity.Id}");
    Console.WriteLine($"\tName:{activity.DisplayName}");
    Console.WriteLine($"\tSource:{activity.Source.Name}");
    Console.WriteLine($"\tSpanId:{activity.TraceId}");
    Console.WriteLine($"\tSpanId:{activity.SpanId}");
    Console.WriteLine($"\tParent:{activity.ParentId}");
    Console.WriteLine($"\tDuration:{activity.Duration}");
    Console.WriteLine($"\tStatus:{activity.Status}");

    Console.WriteLine($"\tAttributes:");
    foreach ((string key, string? value) in activity.Tags)
    {
        Console.WriteLine($"\t\t{key}\t{value}");
    }

    Console.WriteLine($"\tEvents:");
    foreach (var activityEvent in activity.Events)
    {
        Console.WriteLine($"\t\tName:{activityEvent.Name}");
        Console.WriteLine($"\t\tTimestamp:{activityEvent.Timestamp}");
        Console.WriteLine($"\t\tAttributes:");
        foreach ((string key, object? value) in activityEvent.Tags)
        {
            Console.WriteLine($"\t\t\t{key}\t{value}");
        }
    }
}
```

To enable it, you need to add it to the activity source:

```csharp
ActivitySource.AddActivityListener(consoleListener);
```

It will produce logs like this:

```
Trace:00-74c8a413e656fd3feb7511a5cd0b156c-26eca8f5ee9d7994-00
    Name:Do Work
    Source:Tracing.Sample
    Duration:00:00:00.0000683
```

And if you debug the application, the `activity` in the `DoWorkAsync` method is no longer `null`.

In the above example, the `ShouldListenTo` action always returns `true`. This means the listener is going to listen to every activity. If you want to be selective, you can change the action to suit your needs:

```csharp
consoleListener.ShouldListenTo = source => source.Name == "Tracing.Sample";
```

The `Sample` action always returns `AllData` which means the activity will include all attributes, links, events and propagation information. But you could change it to `None` if certain criteria are met, meaning these activities will not be logged.

The `ActivityStopped` action is a callback that is called when an activity stops. This allows us to see the last state the activity is in, prior to it being disposed. You can record as much or as little of the activity as necessary. Tags, links and events are all available to be logged.

### Summary

Instrumenting your code with `ActivitySource` and `Activity` objects can be a bit confusing at first because without a listener, nothing gets recorded! Thankfully, adding an `ActivityListener` is pretty straightforward. The listener gives you a lot of control to record as much or as little as you want!
