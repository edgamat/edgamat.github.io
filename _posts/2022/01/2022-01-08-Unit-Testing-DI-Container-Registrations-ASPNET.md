---
layout: post
title: 'Unit Testing Dependency Injection Container Registrations in ASP.NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Have you ever thought of unit testing the code used to register services in your ASP.NET application? It's not a bad idea. Let's see how.
 
<!--more-->

### Service Registration

In the default ASP.NET templates, pre .NET 6, services were registered in the `ConfigureServices` method of the `Startup` class. .NET 6 introduced minimal API templates which did away with the `Startup` class. Service registrations are sone within the `Program` class. In either case, it can make a lot of sense to 
[create separate methods to register the services for your application]({% post_url 2021/12/2021-12-18-Organize-DI-Registrations %}).

For example, let's assume your application has a service called `ITimeProvider` that is registered as a singleton. One option would be to create an extension method to perform the registration:

```csharp
public static class DomainServiceCollectionExtensions
{
    public static IServiceCollection AddDomainServices(this IServiceCollection services, IConfiguration configuration)
    {
        services.AddSingleton<ITimeProvider, TimeProvider>();
        
        // Other services registered here
        
        return services;
    }
}
```

### Adding a Unit Test

On more than one occasion, I have completed a pull request, with all my unit tests passing, only to find that I had an incorrect (or missing) service registration. I want to get into the habit of creating unit tests to ensure the services are registered as I would expect. The method needs two input parameters; an implementation of the `IServiceCollection` interface and an implementation of the `IConfiguration` interface. 

The `ServiceCollection` class is the implementation of the `IServiceCollection` we can use for our test (and is what the ASP.NET framework uses). And you can use a `ConfigurationBuilder` to create an instance of the `IConfiguration` interface:

```csharp
var services = new ServiceCollection();

var configuration = new ConfigurationBuilder().Build();
```

Now we can write a test for the registration of the `ITimeProvider` service:

```csharp
public void ITimeProvider_Registered_As_Singleton()
{
    // Arrange
    var services = new ServiceCollection();

    var configuration = new ConfigurationBuilder().Build();

    // Act
    services.AddDomainServices(configuration);

    // Assert
    var _ = Assert.Single(services, x => 
        x.ServiceType == typeof(ITimeProvider) && 
        x.Lifetime == ServiceLifetime.Singleton);
}
```

### Testing Conditional Registrations

Some registrations are only made under certain conditions. A typical example is the use of feature toggles. 
Let us assume that the ASP.NET application has a hosted service that runs in the background. It should be registered 
if the configuration includes the following feature toggle in the `appsettings.json`:

```json
{ 
  "EnableBackgroundService": true 
}
```

Here is the updated registration logic:

```csharp
public static class DomainServiceCollectionExtensions
{
    public static IServiceCollection AddDomainServices(this IServiceCollection services, IConfiguration configuration)
    {
        services.AddSingleton<ITimeProvider, TimeProvider>();

        var enableBackgroundService = configuration.GetValue<bool>("EnableBackgroundService");
        if (enableBackgroundService) 
        {
            services.AddHostedService<MyBackgroundService>();
        }
        
        // Other services registered here
        
        return services;
    }
}
```

We can arrange our test using an in-memory configuration source, rather than using a json file:

```csharp
public void MyBackgroundService_Registered_As_Singleton()
{
    // Arrange
    var services = new ServiceCollection();

    var initialData = new Dictionary<string, string>()
    {
        {"EnableBackgroundService", "true"}
    };

    var configuration = new ConfigurationBuilder()
        .AddInMemoryCollection(initialData)
        .Build();

    // Act
    services.AddDomainServices(configuration);

    // Assert
    var _ = Assert.Single(services, x => 
        x.ServiceType == typeof(MyBackgroundService) && 
        x.Lifetime == ServiceLifetime.Singleton);
}
```

And don't forget to include the corresponding test if the service should not be registered:

```csharp
public void MyBackgroundService_Not_Registered()
{
    // Arrange
    var services = new ServiceCollection();

    var initialData = new Dictionary<string, string>()
    {
        {"EnableBackgroundService", "false"}
    };

    var configuration = new ConfigurationBuilder()
        .AddInMemoryCollection(initialData)
        .Build();

    // Act
    services.AddDomainServices(configuration);

    // Assert
    Assert.Empty(services.Where(x => x.ServiceType == typeof(MyBackgroundService)));
}
```

### Summary

I think it wise to add unit tests for the services being registered with an application's Dependency Injection Container. It
can reveal some pretty interesting bugs. The basic registration scenarios are straightforward to test. As the registrations become
more complex, the tests become more complex as well. But it is not impossible, especially if the registrations are refactored into
separate classes.



