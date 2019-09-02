---
layout: post
title: 'Dependency Injection in ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Dependency Injection (DI) was something I read about several years ago and I didn't really understand it well until I encountered ASP\.NET MVC. Specifically, the way [Autofac][autofac] made it all work. I had used other DI containers, but it was using Autofac that really made it clear to me.

<!--more-->

I also found a great resource to help me understand it further. The book was [Dependency Injection in .NET][dibook] by Mark Seemann. Recently a second edition was published titled [Dependency Injection Principles, Practices, and Patterns][dibook2]. I was curious to see how 8 years of experience between these publications had influenced the author's view of DI.

The second edition is more than simply a DI book. It contains great advice on good object-oriented design, and how to write loosely-coupled code.

So how does this fit into ASP.NET Core?

Over the past year I have worked on a few ASP.NET Core REST API services and they all used DI to compose the application. I learned a lot and wanted to write some of these ideas down for future reference. Here goes nothing.

## Microsoft's DI Container

I recommend you read the [Microsoft documentation][ms-di] to learn about how the Microsoft reasons about Dependency Injection
in ASP.NET Core applications.

The built-in DI container in ASP.NET Core is simple to use and configure. It implements three lifetime styles that allow you to implement a solid DI framework:

| Name      | Description                                                                                                        |
| --------- | ------------------------------------------------------------------------------------------------------------------ |
| Transient | Objects are created each time they're requested                                                                    |
| Singleton | Objects are created the first time they're requested and a single instance exists until the application shuts down |
| Scoped    | Objects are created once per request                                                                               |

To configure the DI Container in an ASP.NET Core application, the framework provides a series of extension methods you can use
within the `ConfigureServices` routine of the `Startup` class.

Here is a simple example of how to configure the DI Container. You have an interface and a concrete implementation of the interface you want to inject
into controllers at run-time:

```csharp
public interface ITimeProvider
{
    DateTime Now { get; }
}

public class DefaultTimeProvider : ITimeProvider
{
    public DateTime Now { get { return DateTime.Now; } }
}
```

To inject the concrete class, use the following syntax:

```csharp
services.AddSingleton<ITimeProvider, DefaultTimeProvider>();
```

## Enter Autofac

So why would you want to use some thing more advanced like Autofac?

- Autofac has been around for quite awhile and you amy want to reuse some existing code based on Autofac from a .NET Framework project.
- Or perhaps you have an advanced use case not easily covered by the Microsoft DI container (e.g. a multitenant
  web application, property-injection, auto-registration).
- Autofac has several features that make handling a large number of component registrations easier to maintain (via the use of modules)
- Autofac uses a declarative syntax that may make the registrations easier to read and understand.

Honestly, sometimes it boils down to personal preference of the team developing the application.

Autofac's syntax for registering services is straightforward. Using the same example above, the syntax
is:

```csharp
public void ConfigureContainer(ContainerBuilder builder)
{
    builder.RegisterType<DefaultTimeProvider>().As<ITimeProvider>().SingleInstance();
}
```

NOTE: The Autofac integration uses a separate routine `ConfigureContainer` to configure the DI Container.

I would encourage you to read more about [Autofac integration with ASP.NET Core][auto-asp]. I think
you will find it interesting and may be useful on your next project.

[autofac]: https://autofac.org/
[dibook]: https://www.manning.com/books/dependency-injection-in-dot-net
[dibook2]: https://www.manning.com/books/dependency-injection-principles-practices-patterns
[ms-di]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
[auto-asp]: https://autofac.readthedocs.io/en/latest/integration/aspnetcore.html
