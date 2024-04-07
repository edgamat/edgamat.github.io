---
layout: post
title: 'Recording Exceptions with Diagnostic Activities in .NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

The Diagnostics API in .NET provides a great way to record activities in your code using the new `ActivitySource` and `Activity` objects. But how do you record an exception?

<!--more-->

In the previous post I showed how you could listen to activities and log them to the console.

But what is an exception occurs? How should you record that detail as part of an activity?

Here is an example of an activity in use:

```csharp
public static readonly ActivitySource Source = new("Tracing.Sample", "1.0.0");

private async Task DoWorkAsync(CancellationToken stoppingToken)
{
    using var activity = Source.StartActivity(DiagnosticsNames.DoWork);

    var workId = Guid.NewGuid().ToString();
    activity?.SetTag("work-id", workId);
    ...
    // Do Some Work
    ...
}    
```

Suppose an exception occurs? The OpenTelemetry protocol recommends setting the status of the activity to 'Error', which can be done using the `SetStatus` method:

```csharp
    using var activity = Source.StartActivity(DiagnosticsNames.DoWork);

    try
    {
        var workId = Guid.NewGuid().ToString();
        activity?.SetTag("work-id", workId);
        ...
        // Do Some Work
        ...
    }
    catch (Exception)
    {
        activity?.SetStatus(ActivityStatusCode.Error);
        throw;
    }
}
```

If you want to record the exception details as part of the activity, the OpenTelemetry protocol has a recommendation for that as well:

[https://opentelemetry.io/docs/specs/semconv/exceptions/exceptions-spans/#attributes](https://opentelemetry.io/docs/specs/semconv/exceptions/exceptions-spans/#attributes)

It states that an exception should be recorded as an event with the name `exception`. Many of the language-specific APIs include a `recordException` method for this purpose. The Diagnostics API in .NET does not include such a method. However, inspecting the OpenTelemetry library for .NET, one can see the shim they use for this missing feature. It adds an event with the name `exception` and if provided, includes tags for:

- `exception.type`
- `exception.stacktrace`
- `exception.message`

We can mimic an extension method for this behavior as well:

```csharp
public static class ActivityExtensions
{
    public static void RecordException(this Activity activity, Exception ex, bool hasEscaped = true)
    {
        var tags = new ActivityTagsCollection
        {
            { "exception.type", ex.GetType().FullName },
            { "exception.message", ex.Message },
            { "exception.stacktrace", ex.ToString() },
            { "exception.escaped", hasEscaped }
        };

        var exceptionEvent = new ActivityEvent("exception", default, tags);
        
        activity.AddEvent(exceptionEvent);
        activity.SetStatus(ActivityStatusCode.Error);
    }
}
```

Now we can use it in our `catch` block:

```csharp
    using var activity = Source.StartActivity(DiagnosticsNames.DoWork);

    try
    {
        var workId = Guid.NewGuid().ToString();
        activity?.SetTag("work-id", workId);
        ...
        // Do Some Work
        ...
    }
    catch (Exception)
    {
        activity?.RecordException(ex, true);
        throw;
    }
}
```

And the listener captures the exception details:

```
Trace:00-aebc8545cc9e56e90a535a36839c7fc8-2de822fca0a2818c-01
        Name:Do Work
        Source:Tracing.Sample
        SpanId:aebc8545cc9e56e90a535a36839c7fc8
        SpanId:2de822fca0a2818c
        Parent:
        Duration:00:00:00.0303756
        Status:Error
        Attributes:
                work-id 793b1373-44e0-46f1-bfb1-4f9ff7c20c8f
        Events:
                Name:exception
                Timestamp:4/7/2024 12:04:58 PM +00:00
                Attributes:
                        exception.type  System.Exception
                        exception.message       My Exception
                        exception.stacktrace    System.Exception: My Exception
   at Tracing.Worker.DoWorkAsync(CancellationToken stoppingToken) in C:\Users\edgamat\projects\Tracing\Worker.cs:line 37
                        exception.escaped       True
```

**NOTE** If you are using the OpenTelemetry NuGet package, then use its `RecordException` extension method, no reason to write your own!

### Summary

When instrumenting your code with custom activities, make sue you handle exception appropriately. Follow the OpenTelemetry standards, as they make a lot of sense and are easily to use consistently.
