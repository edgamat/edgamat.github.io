---
layout: post
title: 'A Simple Messaging Library for Azure Service Bus'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I've wanted to learn more about Azure Service Bus for awhile. Let's give it a look.

<!--more-->

### Basic Messaging

I have use several messaging solutions with .NET: Kafka, Google Cloud Pub/Sub, and RabbitMQ. For
RabbitMQ, I used the MassTransit library as an abstraction for my .NET. For Kafka and the Google Cloud
messaging, I create custom messaging libraries, inspired by the design of MassTransit. I found that
the large majority of the features in MassTransit were never used. We only ever use the basics:

- Publish a command message and have competing consumers process the messages.
- Publish an event and have multiple subscribers consume the messages.
- Use a dead-letter queue (DLQ) to handle failed messages.
- All message are published in JSON.

I wanted to explore Azure Service Bus by creating a simple messaging library. The API would be
something like this:

- Create a consumer class that inherits from a base interface (`IMessageConsumer`). This interface exposes
  a generic `ConsumeAsync` method that accepts the object (deserialized from the JSON in the message)
- Register the class as the consumer of a message from a queue or,
- Register the class as the consumer of a message from a subscription,
- If a runtime exception occurs in the consumer, the message is sent to the dead-letter queue
- Create a publisher class that inherits from a base `IMessagePublisher` interface. This interface
  exposes a generic `PublishAsync` method that accepts the name of the topic and the object being
  published

Each type of consumer is registered to a specific queue or subscription:

```csharp
services.AddAzureServiceBus()
    .WithConfiguration(services.Configuration)
    .AddBusConsumersHostedService()
    .Build();
```

The `MyQueueMessageConsumer` inherits from `IMessageConsumer` interface:

```csharp
interface IConsumer<in TMessage> where TMessage: class
{
    Task ConsumeAsync(TMessage message, CancellationToken cancellationToken);
}
```

### Azure Service Bus SDK

According to the current documentation, "Azure Service Bus is a fully managed enterprise message
broker with message queues and publish-subscribe topics." It supports queues, topics and subscriptions.
The NuGet package you will require is:

```bash
dotnet add package Azure.Messaging.ServiceBus -v7.20.1
```

For my concept of basic messaging, I need to configure a `ServiceBusClient` that is used to create
the objects needed to publish and consume messages. One of the `ServiceBusClient` constructors accepts
a connection string, which is how I have chosen to configure the client. Another constructor accepts
a namespace and a credentials object. This can be used with Azure Identity to authenticate each request.

I chose to use the `Microsoft.Extensions.Azure` library to register the `ServiceBusClient`:

```csharp
var settings = new AzureServiceBusSettings();
configuration.GetSection("AzureServiceBus").Bind(settings);
services.AddSingleton(settings);

_services.AddAzureClients(builder =>
{
    builder.AddServiceBusClient(settings.ConnectionString);

    foreach (var queueName in settings.Queues.Where(q => q.Enabled).Select(q => q.QueueName))
    {
        builder.AddClient<ServiceBusSender, ServiceBusClientOptions>((_, _, provider) =>
            provider
                .GetRequiredService<ServiceBusClient>()
                .CreateSender(queueName)
        ).WithName(queueName);
    }
});
```

The `ServiceBusClient` and `ServiceBusSender objects are designed to be thread-safe and long-lived,
so they can be created as a singleton and used throughout the lifetime of your application.
I'll create a background hosted service to use the client to process the messages.

In addition to the connection string (which you can get from the Azure Portal), I have chosen to configure
the following in the `appsettings.json`:

- The name of each queue, and the maximum number of consumers for the queue
- The name of the topic and subscription, and the maximum number of consumers for the subscription
- A flag that can enable/disable each of the queues/subscriptions

```json
{
  "AzureServiceBus": {
    "ConnectionString": "<From Azure Portal>",
    "Queues": [
      {
        "QueueName": "queue.1",
        "MaxCompetingConsumers": 4,
        "ConsumerType": "Edgamat.Messaging.Samples.Host.MyQueueConsumer",
        "Enabled": true
      }
    ],
    "Subscriptions": [
      {
        "TopicName": "topic.1",
        "SubscriptionName": "subscription.1",
        "MaxCompetingConsumers": 4,
        "ConsumerType": "Edgamat.Messaging.Samples.Host.MySubscriptionConsumer",
        "Enabled": true
      }
    ]
  }
}
```

### Publishing Messages

We want a simple means to publish messages to a queue or a topic.

```csharp
public class JsonPublisher : IPublisher
{
    private readonly IAzureClientFactory<ServiceBusSender> _factory;

    public JsonPublisher(IAzureClientFactory<ServiceBusSender> senderFactory)
    {
        _factory = senderFactory;
    }

    public async Task<string> PublishAsync<T>(string queueOrTopicName, T message, CancellationToken cancellationToken = default) where T : class
    {
        var sender = _factory.CreateClient(queueOrTopicName);

        var jsonMessage = JsonSerializer.Serialize(message);
        var serviceBusMessage = new ServiceBusMessage(jsonMessage)
        {
            ContentType = "application/json",
            MessageId = Guid.NewGuid().ToString(),
        };

        await sender.SendMessageAsync(serviceBusMessage, cancellationToken);

        return serviceBusMessage.MessageId;
    }
}


var publisher = scopedServices.GetRequiredService<IPublisher>();

var messageId = await publisher.PublishAsync("queue.1", new MyMessage(1, 1));
```

### Consuming Messages

Using the queue settings, we register each of the consumer classes using a `Scoped` lifetime:

```csharp
foreach (var queue in settings.Queues.Where(q => q.Enabled))
{
    var consumerType = Type.GetType(queue.ConsumerType)
        ?? throw new Exception($"Could not find consumer type '{queue.ConsumerType}' for queue '{queue.QueueName}'");

    if (!typeof(IConsumer<MessageContext>).IsAssignableFrom(consumerType))
      throw new Exception($"Consumer type '{consumerType.FullName}' does not implement the IConsumer<MessageContext> interface.");

    builder.Services.AddScoped(consumerType);
}

foreach (var subscription in settings.Subscriptions.Where(q => q.Enabled))
{
    var consumerType = Type.GetType(queue.ConsumerType)
        ?? throw new Exception($"Could not find consumer type '{queue.ConsumerType}' for subscription '{subscription.SubscriptionName}'");

    if (!typeof(IConsumer<MessageContext>).IsAssignableFrom(consumerType))
      throw new Exception($"Consumer type '{consumerType.FullName}' does not implement the IConsumer<MessageContext> interface.");

    builder.Services.AddScoped(consumerType);
}
```

### The Secret Sauce

The secret to making this work is how the queues are processed. For this I wrote a background service
that creates separate processors for each queue and each subscription. The service injects the
`ServiceBusClient` and the service provider. The client is used to create the queue/subscription processors,
and the provider is used to create the configured consumer classes.

```csharp
public class ServiceBusConsumersHostedService : IHostedService
{
    private readonly ServiceBusClient _client;
    private readonly IServiceProvider _provider;
    private readonly List<ServiceBusProcessor> _processors = [];

    public ServiceBusConsumersHostedService(
        ServiceBusClient client,
        IServiceProvider provider)
    {
        _client = client;
        _provider = provider;
    }
    ...
}    
```

In the `StartAsync` method, we initialize each of the queue processors:

```csharp
var settings = provider.GetRequiredService<AzureServiceBusSettings>();

foreach (var queue in settings.Queues.Where(q => q.Enabled))
{
    var processor = _client.CreateProcessor(queue.QueueName, new ServiceBusProcessorOptions
    {
        MaxConcurrentCalls = queue.MaxCompetingConsumers,
        AutoCompleteMessages = false
    });

    processor.ProcessMessageAsync += async args =>
    {
        var messageContext = new MessageContext(queue, args.Message);

        using var scope = _provider.CreateScope();

        var consumerType = Type.GetType(queue.ConsumerType);

        var consumer = (IConsumer<MessageContext>)scope.ServiceProvider.GetRequiredService(consumerType);

        await consumer.ConsumeAsync(messageContext, args.CancellationToken);

        await args.CompleteMessageAsync(args.Message, args.CancellationToken);
    };

    _processors.Add(processor);

    await processor.StartProcessingAsync(cancellationToken);
}
```

The same initialization is performed for each of the subscriptions:

```csharp
foreach (var subscription in settings.Subscriptions.Where(q => q.Enabled))
{
    var processor = _client.CreateProcessor(
      subscription.TopicName, 
      subscription.SubscriptionName, 
      new ServiceBusProcessorOptions
    {
        MaxConcurrentCalls = queue.MaxCompetingConsumers,
        AutoCompleteMessages = false
    });

    processor.ProcessMessageAsync += async args =>
    {
        var messageContext = new MessageContext(subscription, args.Message);

        using var scope = _provider.CreateScope();

        var consumerType = Type.GetType(subscription.ConsumerType);

        var consumer = (IConsumer<MessageContext>)scope.ServiceProvider.GetRequiredService(consumerType);

        await consumer.ConsumeAsync(messageContext, args.CancellationToken);

        await args.CompleteMessageAsync(args.Message, args.CancellationToken);
    };

    _processors.Add(processor);

    await processor.StartProcessingAsync(cancellationToken);
}
```

The `MessageContext` stores the details about the current queue and message:

```csharp
public class MessageContext
{
    public string SourceName { get; set; } = string.Empty;

    public BinaryData RawPayload { get; set; } = default!;

    public string MessageId { get; set; } = string.Empty;

    public string CorrelationId { get; set; } = string.Empty;

    public int DeliveryAttempt { get; set; }

    public int MaxDeliveryAttempts { get; set; }
}
```

Note that the processor is configured with `AutoCompleteMessages` = `false`. Since we only complete
the message when no exceptions occur during its consumption, the message will automatically be
retried when exceptions occur. Once it reaches the maximum number of allowed retries, it will be
sent to the configured dead letter queue.

In the `StopAsync` method, we dispose each of the queue processors:

```csharp
public async Task StopAsync(CancellationToken cancellationToken)
{
    // Stop and dispose all processors
    foreach (var p in _processors)
    {
        await p.StopProcessingAsync(cancellationToken);
        await p.DisposeAsync();
    }
}

```

### Consuming JSON Messages

Our consumer is expecting the 'raw' message to be a JSON document. To make things easier to consume,
we want to convert the raw JSON into an object. For this be create an abstract JSON consumer we
can inherit from:

```csharp
public abstract class JsonConsumer<T> : IConsumer<MessageContext> where T : class
{
    public Task ConsumeAsync(MessageContext messageContext, CancellationToken token)
    {
        MessageContext = messageContext;

        var message = messageContext.RawPayload.ToObjectFromJson<T>() ?? 
            throw new JsonException("Unable to deserialize message body.");

        return ConsumeMessageAsync(message, token);
    }

    public MessageContext? MessageContext { get; set; }

    public abstract Task ConsumeMessageAsync(T message, CancellationToken cancellationToken);
}
```

Our consumer class now has all the details of Azure Service Bus removed:

```csharp
public record MyMessage(int Batch, int Message);

public class MyMessageConsumer : JsonConsumer<MyMessage>
{
    private readonly ILogger<MyMessageConsumer> _logger;

    public MyMessageConsumer(ILogger<MyMessageConsumer> logger) : base(logger)
    {
        _logger = logger;
    }

    public override Task ConsumeMessageAsync(MyMessage message, CancellationToken token)
    {
        _logger.LogInformation("Consuming message: {Batch} - {Message}", message.Batch, message.Message);

        return Task.CompletedTask;
    }
}
```

### Summary

I have wanted to learn how to user Azure Service Bus for awhile now. Not all its features, but just the
basics for now. The publishers and consumers I have created make using the service bus in an application
straightforward. I have hidden the details of how to configure and register the objects behind simple
to use abstractions. I had a lot of fun creating these samples.
