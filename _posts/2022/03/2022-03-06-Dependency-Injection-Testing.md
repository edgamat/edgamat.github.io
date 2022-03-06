---
layout: post
title: 'Dependency Injection Part 3 - Testing .NET DI Container Registrations'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In this 3rd article on Dependency Injection I want to show how you can write unit tests to verify the services have been properly configured in the DI Container.

<!--more-->

On my current project there have been times when the cause of a bug was due to an incorrect registration of services in the DI Container. We hadn't included any unit tests to verify the registrations were correct. In some cases, the registrations were missing entirely! I set out to write some unit tests to help protect us from this sort of thing.

### Verifying the Registration

The registration of services  occurs in `ConfigureServices` method if the `Startup.cs` class:

```csharp
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddDomainServices(Configuration);
        services.AddControllers();
        ...
    }
}
```

In this example, `AddDomainServices` is an extension method that registers my domain-level services:

```csharp
services.AddScoped<IMyScopedService, MyScopedService>();
services.AddSingleton<IMySingletonService, MySingletonService>();
```

I would like to write unit tests that verify that the `IMyScopedService` service is registered with a scoped lifetime

### Arrange

The inputs to the `AddDomainServices` are an implementation of the `IServiceCollection` and an implementation of the `IConfiguration`. Here is a sample of how to setup in-memory versions of these inputs:

```csharp
// Arrange
var initialData = new Dictionary<string, string>
{
    {"Key", "Value"},
};

var configuration = new ConfigurationBuilder()
    .AddInMemoryCollection(initialData)
    .Build();

var services = new ServiceCollection();
``` 

The `initialData` contains all the configuration data needed during registration. For example, suppose the registration of services is conditionally based on configuration data

```csharp
var enableScopedService = configuration.GetValue("MyApp:EnableScopedService", false);
if (enableScopedService)
{
    services.AddScoped<IMyScopedService, RealScopedService>();
}
else
{
    services.AddScoped<IMyScopedService, DummyScopedService>();
}
```

Then you would provide it in the `initialData` collection:

```csharp
var initialData = new Dictionary<string, string>
{
    {"MyApp:EnableScopedService", "true"},
};
```

### Act

Once you have setup the inputs, you can then call the registration method:

```csharp
// Arrange
...

// Act
var sut = new Startup(configuration);

sut.ConfigureServices(services);
```

Why not test only the registrations in the `AddDomainServices` method? These services may rely on other services registered in other classes, so it is important registrations, not just some of them.

### Assert

Here are some of the assertions you can make:

```csharp
// The service is registered (yay!)
var serviceDescriptor = Assert.Single(services, x => x.ServiceType == typeof(IMyScopedService));

// The service is registered with something useful
Assert.NotNull(serviceDescriptor);

// The service is registered with the expected implementation
Assert.Equal(typeof(RealScopedService), serviceDescriptor.ImplementationType);

// The service is registered with the expected lifetime (lifestyle)
Assert.Equal(ServiceLifetime.Scoped, serviceDescriptor.Lifetime);
```

### Summary

It can save a lot of time if you write unit tests to verify that classes are registered properly. Someone may want to change the configured lifetime of a service. With a unit test in place, it can let the developer know that changing the lifetime will affect how the code behaves. It is also important to test any conditional logic, or service registrations based on the configuration data. Don't forget to include these tests (or better yet, write them first and then update the DI Container registrations).
