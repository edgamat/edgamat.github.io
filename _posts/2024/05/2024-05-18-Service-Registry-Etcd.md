---
layout: post
title: 'Using etcd as a Service Registry from C#'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Using a service registry can give many benefits. I'd like to see how we can use etcd for this purpose.

<!--more-->

At a previous client, we use Netflix's Eureka as a service registry. It allow multiple instances of the same service to registry themselves and coordinate work between them. One of the instances would be elected the leader and would be responsible for special tasks, like running data migrations, polling for new work to be added to queues, and a few other jobs. I was always curious to know how difficult other services, like etcd, would be to use for this purpose. Let's find out.

### Running etcd Locally

Step one is setting up an instance of etcd to test with. For that I'll use a docker compose file and run a container locally:

```yaml
services:
  etcd:
    image: bitnami/etcd:latest
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_ADVERTISE_CLIENT_URLS=http://etcd:2379
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
    ports:
      - "2379:2379"
      - "2380:2380"
    volumes:
      - C:/DockerVolumes/etcd:/bitnami/etcd
```

### The Registry Service API

To use etcd as a service registry, we need to define an API that we would like etcd to implement for us. The obvious first method would be to register a service instance. The second would be to un-register an instance as the application shuts down. and the 3rd would be to get a list of all the instances currently registered.

Each service instance must identify itself with the service registry. It does wo using the following:
- The name of the service (service name)
- A value that uniquely identifies the instance of the service (service id)
- A URL where the service instance can be accessed (service address)

For example, a service called "OrderShippingService", with an instance hosted at http://localhost:5000, could use these values to identify the instance:

```csharp
var serviceInstanceInfo = new
{
    Id = Guid.NewGuid().ToString(),
    Name = "OrderShippingService",
    Address = "http://localhost:5000"
};
```

### Configure the Application

Let's create a Console application and add a nuget package to interface with etcd:

```powershell
dotnet new console -n EtcdServiceRegistryApp
dotnet add package dotnet-etcd
```

Let's store the service registry details in a settings file:

```json
{
    "ServiceRegistry": {
        "ConnectionString": "http://localhost:2379",
        "ServiceName": "EtcdServiceRegistryApp",
        "ServiceAddress": "http://localhost:8080"
    }
}
```

We can use a class to access these values via the Options framework using the DI Container:

```csharp
public class ServiceRegistryConfiguration
{
    public required string ConnectionString { get; set; }
    public required string ServiceName { get; set; }
    public required string ServiceAddress { get; set; }
}

// Program.cs
builder.Services.AddOptions<ServiceRegistryConfiguration>().Bind(builder.Configuration.GetSection("ServiceRegistry"));
```

### Registering a Service Instance

Let's start creating our Service Registry API. We want to have an EtcdClient and the configuration details provided by the DI Container:

```csharp
public class ServiceRegistry
{
    private readonly IEtcdClient _client;
    private readonly ILogger<ServiceRegistry> _logger;
    private readonly string _instanceId;
    private readonly string _serviceName;
    private readonly string _serviceInstanceKey;
    private readonly string _serviceAddress;
    
    public ServiceRegistry(
        ServiceRegistryConfiguration configuration, 
        IEtcdClient client,
        ILogger<ServiceRegistry> logger)
    {
        _client = client;
        _logger = logger;
        _instanceId = Guid.NewGuid().ToString();
        _serviceName = configuration.ServiceName;
        _serviceAddress = configuration.ServiceAddress;
        _serviceInstanceKey = $"/services/{_serviceName}/{_instanceId}";
    }
}

// Program.cs
builder.Services.AddSingleton<IEtcdClient>(sp =>
{
    var connectionString = sp.GetRequiredService<IOptions<ServiceRegistryConfiguration>>().Value.ConnectionString;
    return new EtcdClient(connectionString);
});
builder.Services.AddSingleton<ServiceRegistry>();
```

We use a GUID to create a unique identifier for the instance. In etcd, we will use the service name and instance id to form a unique key for the service instance.

We can use the PUT endpoint to register the instance:

```csharp
public async Task RegisterServiceAsync(CancellationToken token)
{
    var serviceInstanceInfo = new
    {
        Id = _instanceId,
        Name = _serviceName,
        Address = _serviceAddress,
    };

    var serviceValue = JsonSerializer.Serialize(serviceInstanceInfo);

    await _client.PutAsync(_serviceInstanceKey, serviceValue, null, null, token);
    
    _logger.LogInformation("Successfully registered service {serviceInstanceInfo}", serviceInstanceInfo);
}
```

Similarly we can use the DELETE endpoint to un-register the instance:

```csharp
public async Task UnregisterServiceAsync(CancellationToken token)
{
    await _client.DeleteAsync(_serviceInstanceKey, null, null, token);
    
    _logger.LogInformation("Successfully un-registered service {ServiceKey}", _serviceInstanceKey);
}
```

In the startup pipeline we can now use our new service registry:

```csharp
var app = builder.Build();

var registry = app.Services.GetRequiredService<ServiceRegistry>();

await registry.RegisterServiceAsync(CancellationToken.None);

```

We can use the `ApplicationStopped` cancellation token to unregister the service instance:

```csharp
var lifetime = app.Services.GetRequiredService<IHostApplicationLifetime>();

lifetime.ApplicationStopped.Register(() =>
{
    registry.UnregisterServiceAsync(CancellationToken.None).Wait();
});
```

NOTE: When using the WebApplicationBuilder in an ASP.NET application, the `IHostApplicationLifetime` is exposed via the `Lifetime` property:

```csharp
app.Lifetime.ApplicationStopped.Register(() =>
{
    registry.UnregisterServiceAsync(CancellationToken.None).Wait();
});
```

### Removing Service Instances Automatically

While testing this process I failed to call the `UnregisterServiceAsync` method and remove the key from the registry. What I'd like is for the keys to expire after a specificed timeout, say 30 seconds. That way they will cleanup after themselves. I can do that using a lease. 

```csharp
private long _leaseId;

public async Task RegisterServiceAsync(CancellationToken token)
{
    // Create a lease with a time to live (TTL) of 30 seconds
    var leaseGrantRequest = new LeaseGrantRequest { TTL = 30 };
    var leaseGrantResponse = await _client.LeaseGrantAsync(leaseGrantRequest);
    _leaseId = leaseGrantResponse.ID;
    
    var serviceInstanceInfo = new
    {
        Id = _instanceId,
        Name = _serviceName,
        Address = _serviceAddress,
    };

        var serviceValue = JsonSerializer.Serialize(serviceInstanceInfo);

        var putRequest = new PutRequest
        {
            Key = ByteString.CopyFromUtf8(_serviceInstanceKey),
            Value = ByteString.CopyFromUtf8(serviceValue),
            Lease = _leaseId
        };

        await _client.PutAsync(putRequest, null, null, token);
        
        _logger.LogInformation("Successfully registered service {serviceInstanceInfo} with lease {LeaseId}", serviceInstanceInfo, _leaseId);
```

After 30 seconds the keys will be removed from the etcd system. That means that I'll need to update the lease periodically to let etcd know the instance is still alive.

The `dotnet-etcd` library provides a `KeepLeaseAlive` method for just such a purpose:

```csharp
public async Task SendHeartbeatsAsync(CancellationToken token)
{
    await _client.LeaseKeepAlive(_leaseId, token);
}
```

Create a class that implements the `BackgroundService` and call the `SendHeartbeatsAsync` method:

```csharp
// ServiceRegistryHeartbeatService.cs
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    try
    {
        await _registry.SendHeartbeatsAsync(stoppingToken);
    }
    catch (OperationCanceledException)
    {
        _logger.LogInformation("Stopping Token Cancelled");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Unhandled exception while sending heartbeats");
    }
}
```

Then register this new service:

```csharp
builder.Services.AddHostedService<ServiceRegistryHeartbeatService>();
```

### Summary

While not production-ready, I believe I have worked out how to use etcd as a service registry for .NET applications. In my next post, I'll look at leadership election!

