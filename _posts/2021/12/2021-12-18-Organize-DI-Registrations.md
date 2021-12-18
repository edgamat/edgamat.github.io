---
layout: post
title: 'How to organize service registration with the Microsoft Dependency Injection Container'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Sometimes the number of Dependency Injection services registered by your .NET application can grow to a size that means you should start splitting them out into separate pieces. This post discusses ways you may consider doing that. 

<!--more-->

### The Problem

The Microsoft ASP.NET Core templates (including .NET 5 and 6) provide a means to register services with the built-in Dependency Injection (DI). Prior to the use of Minimal APIs in .NET 6 templates, the standard approach was to add all the service registrations for an application in the `ConfigureServices` method of the `Startup.cs` class. In a typical REST API project, you would find registrations for a number of services:

- Entity Framework Code DbContext classes
- Messaging Libraries (Google Cloud Platform, RabbitMQ, Azure Service Bus, Apache Kafka)
- Health Checks
- Open API (Swagger)

You would also find a series of service registrations for the services within your application domain. And this is where I'd like to focus some attention on.

As your domain logic grows you may have a series of services over a number of namespaces (for example, you may have a separate namespace for each aggregate in your domain). Before you know it, the number of using directives at the top of your `Startup.cs` class will grow. In addition, the `ConfigureServices` method of the `Startup.cs` class can contain a lot of code and it too can be un-manageable. 

A better approach to registering your domain services is to register them in groups.

### The Solution

The ASP.NET Core framework uses a convention to group service registrations. It uses extension methods with the name `Add{GroupName}` to the `IServiceCollection`. For example, here is the extension method used with Entity Framework Core:

```csharp
var connectionString = Configuration.GetConnectionString("DefaultConnection");
services.AddDbContext<ApplicationDbContext>(options => options.UseSqlServer(connectionString));
``` 

The `AddDbContext` extension method encapsulates all the logic necessary to register the dependencies for Entity Framework.

When it comes to registering your domain services, you should take the same approach. Rather than registering the services individually within the `ConfigureServices` method, you should register them all as a group.


### Registering Groups of Services

To create your own extension method, create a separate class in your project:

```csharp
namespace Microsoft.Extensions.DependencyInjection
{
    public static class DomainServiceCollectionExtensions
    {
        public static IServiceCollection AddDomainServices(this IServiceCollection services, IConfiguration configuration)
        {
            // Register your domain services here
            
            return services;
        }
    }
}
```

Now you can register all the domain services with one line of code in your `Startup` class (or `Program.cs` if using Minimal APIs):

```csharp
// Startup.cs ConfigureServices in .NET 5
services.AddDomainServices(Configuration);

// Program.cs in .NET 6 using minimal API template
builder.Services.AddDomainServices(builder.Configuration);
```

I hope it is clear that this approach makes it much easier to understand the flow through the service registration 'pipeline' in the Startup/Program classes. Removing these registrations to a separate file makes them much easier to manage.

Here is the primary web page for information on using Dependency Injection with ASP.NET:

<a href="https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection" target="_blank">https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection</a>

You will find it gets updated from time to time (especially when a new version of .NET is released). 

In the section "Register groups of services with extension methods", it advises to place the extension methods in the `Microsoft.Extensions.DependencyInjection` namespace, as I have demonstrated above. It improves Intellisense and reduces the need to add individual namespaces in the Startup/Program classes for each group of service registrations. You may find that JetBrains Resharper (or other code analysis tools) complain that the namespace does not match the folder structure. But it is easy enough to suppress this warning.

### Final Thoughts

What I have represented here is a simple example. But I think it is still a good starting place. If your service registrations of your domain classes becomes much more complex, you may need to divide the `AddDomainServices` method into smaller groups. 

There are often interdependencies between groups of services that make having the registration of the domain services in a separate file 'inconvenient', compared to having the services registered in the Startup/Program classes. If this occurs, you would be wise to refactor the service registrations to minimize or eliminate these interdependencies as much as possible, just like you would with the interdependencies in your domain services. This is an opportunity to make you code simpler, so take advantage of it!

