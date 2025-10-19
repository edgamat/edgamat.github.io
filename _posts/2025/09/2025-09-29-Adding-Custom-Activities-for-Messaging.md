---
layout: post
title: 'Adding Custom Traces for OpenTelemetry'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I have been writing a simple messaging library layered on top of Azure Service Bus. I want to add
my own custom traces that OpenTelemetry can export.

<!--more-->

### Preference for Custom Diagnostics

Recent versions of the Azure Service Bus SDK in .NET include built-in diagnostics. The SDK creates
diagnostic activities in order to generate telemetry data you can export to external monitoring
tools, like Azure Application Insights.

Here is a list of the operations the SDK tracks: [https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-end-to-end-tracing?tabs=net-standard-sdk-2#instrumented-operations](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-end-to-end-tracing?tabs=net-standard-sdk-2#instrumented-operations)

The diagnostics built into the SDK are good for understanding how Azure Service Bus is behaving,
but not how my application is behaving. I would rather use custom diagnostics for my messaging library.  

Specifically, I want to know:

- When a message is published
- When a message is consumed
- When a failure to consume a message causes a retry
- When a failure to consume a message causes a message to be sent to a dead letter queue

### A Common Source

I want to ensure all traces are associated with a common source. To do that, I'll set up the `ActivitySource`
for everything to use:

```csharp
public static class DiagnosticsConfig
{
    public const string ServiceName = "Edgamat.Messaging";

    public static ActivitySource Source { get; } = new(ServiceName);
}
```

### Publishing

When publishing a message, I want to create an activity, and flag it as a `Producer` kind:

```csharp
using var activity = DiagnosticsConfig.Source.StartActivity("MessagePublisher.Publish", ActivityKind.Producer);
```

I want to also include the details of the current message. I did that using tags:

```csharp
activity?.SetTag("messaging.system", "edgamat.azureservicebus");
activity?.SetTag("messaging.destination.name", queueOrTopicName);
activity?.SetTag("messaging.message.type", typeof(T).FullName);
```

After the message is published, I also add the message id to the activity:

```csharp
activity?.SetTag("messaging.message.id", message.MessageId);
```

### Distributed Tracing

I want to track messages from the producer to the consumer, especially if the producer and consumer are
in different applications. The latest versions of the Azure Service Bus already provide this capability,
based on the [W3C Trace-Context](https://www.w3.org/TR/trace-context/). In the message application
properties, there is a property named `Diagnostic-Id` following the W3C Trace-Context trace parent header format.
I can use this to track messages. If this wasn't available, I would need to provide my own:

```csharp
serviceBusMessage.ApplicationProperties["X-Traceparent"] = activity?.Context.TraceId.ToString();
```

### Consuming

When consuming a message, I want to create an activity, and flag it as a `Consumer` kind:

```csharp
var activity = DiagnosticsConfig.Source.StartActivity("MessageConsumer.Consume", ActivityKind.Consumer);
```

But I also need to link this activity back to the activity created when the message was produced. I
can use the `Diagnostic-Id` to create a parent context:

```csharp
ActivityContext parentContext = default;

if (message.ApplicationProperties.TryGetValue("Diagnostic-Id", out var objectId) && objectId is string diagnosticId)
{
    parentContext = ActivityContext.TryParse(diagnosticId, null, out var parsedContext)
        ? parsedContext
        : default;
}

var activity = DiagnosticsConfig.Source.StartActivity("MessageConsumer.Consume", ActivityKind.Consumer, parentContext);
```

Next, I'll create tags for the message properties:

```csharp
activity?.SetTag("messaging.system", "edgamat.azureservicebus");
activity?.SetTag("messaging.destination.name", messageContext.QueueOrTopicName);

if (!string.IsNullOrWhiteSpace(messageContext.SubscriptionName))
{
    activity?.SetTag("messaging.destination.subscription.name", messageContext.SubscriptionName);
}

activity?.SetTag("messaging.message.type", typeof(T).FullName);
activity?.SetTag("messaging.message.id", messageContext.MessageId);
activity?.SetTag("messaging.delivery_attempt", messageContext.DeliveryAttempt);
activity?.SetTag("messaging.max_delivery_attempts", messageContext.MaxDeliveryAttempts);
```

### Tracking Exceptions/Retries

The message library has built-in retries on messages. This will get triggered if an exception occurs. In
the exception `catch` block, I add the exception details:

```csharp
try
{
    // Consume message
}
catch (Exception ex)
{
    if (messageContext.DeliveryAttempt >= messageContext.MaxDeliveryAttempts)
    {
        activity?.SetStatus(ActivityStatusCode.Error, "Failure to consume message");
        activity?.AddException(ex);
        throw;
    }

    // add activity for retry delay
    using (var retryActivity = DiagnosticsConfig.Source.StartActivity("RetryDelay", ActivityKind.Internal))
    {
        retryActivity?.SetTag("messaging.retry.delay", messageContext.RetryDelay.TotalMilliseconds);
        await Task.Delay(messageContext.RetryDelay, token);
    }

    throw;
}
```

I can find retries by filtering on the `RetryDelay` activities. I can find message that are sent to
the dead letter queue by filtering on `MessageConsumer.Consume` activities with an error status.

### Summary

Adding the custom diagnostics is not difficult. It provides me with more flexibility when exporting
telemetry data and I can export it all to any OpenTelemetry collector.
