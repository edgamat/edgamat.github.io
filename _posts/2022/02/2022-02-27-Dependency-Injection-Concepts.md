---
layout: post
title: 'Dependency Injection Part 2 - Concepts'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In this second article, let's review some of the core concepts of Dependency Injection (DI).

<!--more-->

In the first article I defined Dependency Injection (DI) as a set of software design principles and patterns that enables you to develop loosely coupled code. It demonstrated the technique of injecting dependencies into a class rather than explicitly coding the dependencies in a class. Here is the _loosley coupled_ sample we reviewed:

```csharp
public class CustomerController
{
    private readonly ILogger<CustomerController> _logger;

    public CustomerController(ILogger<CustomerController> logger)
    {
        _logger = logger;
    }
    
    ...
}
```

Somewhere in the application code there needs to be a place where these objects are created (or composed). A request is made to the application to create an instance of the `CustomerController`. The application needs to know how to create an instance of `CustomerController` (the object) as well as the `ILogger<CustomerController>` interface (the dependency). In a large application there can be dozens or hundreds of such objects and their dependencies. 

### Composite Root

The best practice is to define a logical location in the code where the objects are composed together. This is called the "Composite Root", and should be placed close to the entry point to the application.

In the .NET 5 ASP.NET project templates, the `Startup` class has a method called `ConfigureServices`. This is the Composite Root for the application. All the rules defining what implementation to use for each dependency are defined here using a DI Container. As the name implies, this method in the `Startup` class is called when the application starts.

```csharp
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        // This is the composite root
        services.AddControllers();
    }
}
```

### DI Container

A DI Container is a library that provides dependency injection functionality that automates the task of composing objects, as well as intercepting and managing the lifetime of the objects it creates.

When an ASP.NET application receives a request, it uses the configured routing settings to choose the controller used to process the request. It uses a DI Container to create an instance of the controller and all of its dependencies. 

In our example, the DI container knows that in order to create a `CustomerController`, it needs to have an object that implements the `ILogger<CustomerController>` interface to pass into the constructor of the `CustomerController`. The DI Container finds an instance of this interface to use, and if it can't find one, the DI Container creates an instance.

Microsoft provides a default DI Container, and it allows for third-party DI Containers to be used as well. The `IServiceCollection` parameter of the `ConfigureServices` method represents the list of all the objects the DI Container may be asked to create. In this context a 'service' is a possible dependency that a class may need to be function properly. The DI container includes a series of helper methods to defines how to construct these dependencies and how to manage their lifestyle.

NOTE: The DI Container keeps track of all the objects is has created and disposes of them when they are no longer being used. 

When the DI Container is finished configuring all the services, it creates what is called a "Service Provider" which is used to create (or resolve) objects when they are needed. 

### Object Lifestyle

An object lifestyle defines the intended lifetime of a dependency (when they are created and when they are disposed of). Three lifestyles are typically implemented in most DI Containers:

| Name | Description |
| - | - |
| Transient | Objects are created each time they're requested |
| Singleton | Objects are created the first time they're requested and a single instance exists while the application is running |
| Scoped | Objects are created once per 'scope'. They behave like a singleton within the defined scope |

In an ASP.NET application, a separate scope is created for each web request being processed.

When you register a class with the DI Container, you must define the lifestyle is should use. The built-in DI Container includes methods to define these:

```csharp
services.AddTransient<IMyInterface, MyImplementation>();
services.AddScoped<IMyScopedInterface, MyScopedImplementation>();
services.AddSingleton<IMySingletonInterface, MySingletonImplementation>();
```
**NOTE** A class registered with a Singleton lifestyle does not need to implement the Singleton Design Pattern. It just has to be thread-safe.

Transient and scoped objects are disposed as soon as the scope they are created in is disposed. Singleton objects are disposed of when the application shuts down.

**How do I choose?**

As you can imagine, choosing the proper lifestyle for a dependency is very important. If you choose the wrong lifestyle, it can cause bugs that are typically hard to diagnose.

Singleton dependencies typically have to be thread-safe because there may be many simultaneous calls being made to them from different threads in your application. Taking our earlier example, the implementation of the `ILogger<CustomerController>` is thread-safe. Each web request being processed by the application has access to the same logging implementation. 

Scoped dependencies are a blessing and a curse. They can make some problems easier to solve, but can make the code more difficult to understand and reason about. We'll look at some examples of these scenarios in future articles.



