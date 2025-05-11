---
layout: post
title: 'Abstracting Infrastructure Concerns'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Organizing and creating the architecture for a .NET application can be largely a matter of preference.
How many projects should you use? What code belongs in each project? Let's take a look at
infrastructure concerns.

<!--more-->

### What is Infrastructure?

In my context, _infrastructure_ refers to components that interact with external systems or frameworks,
such as the following:

- Data Access (e.g. EntityFramework Core, Dapper)
- HTTP Requests (e.g. HttpClient)
- Emails (e.g. MimeKit)
- Messaging (e.g. RabbitMQ)
- Caching (e.g. Redis)
- Threading (e.g. Task.WhenAll, Task.Delay)
- System Clock (e.g. DateTime.UtcNow, DateTimeOffset.UtcNow)
- File Systems (e.g. Directory, File)

### Prefer Abstractions

Wherever practical, I try to ensure these components provide their functionality via abstractions. This
allows for the core logic that uses these components to be unaware of how they are implemented. This
reduces the coupling between the external components and the core (internal) components of an
application. This makes the code more complex, but also makes it easier to test and change over time.
This is a tradeoff I am willing to make for all but the most trivial of applications.

Let's take a look at a typical Web API solution. I would expect to find (at a minimum) the following:

- A Web API (ASP.NET Core) project for the API endpoints to be configured
- A Class Library project for all my core (domain) logic specific to the application
- A Class Library project for all my infrastructure concerns

A simple scenario is an API that interacts with a database. In this case, the Infrastructure
project might have a dependency on Entity Framework Core or Dapper. You may create this solution:

```text
MyApp.sln
=> MyApp.Api
=> MyApp.Core
=> MyApp.Data
```

Depending on your architecture, the project dependencies may be:

```text
// N-tier Architecture
MyApp.Api => MyApp.Core => MyApp.Data
```

or

```text
// Clean Architecture
MyApp.Api => MyApp.Core <= MyApp.Data
```

Over time, you may include more infrastructure, say accessing a 3rd party API. You have a few options
for incorporating the new external dependency. One option would be to introduce a new project:

```text
MyApp.sln
=> MyApp.Api
=> MyApp.Core
=> MyApp.Data
=> MyApp.ThirdPartyApi
```

Or you may refactor the code into a new `Infrastructure` project, using separate directories for the
different external services:

```text
MyApp.sln
=> MyApp.Api
=> MyApp.Core
=> MyApp.Infrastructure
    => Data
    => ThirdPartyApi
```

Either approach is fine. Your application will grow and you will find a way to organize the code
as it grows.

### Where do the Abstractions Live?

In my opinion, all the external systems should only be accessed via abstractions, typically
interfaces in .NET. The components in the infrastructure project are implementations of those
abstractions. Let's say you have a domain entity called `Widget` and you require a means to persist
it to a database and query the contents of this database. You could introduce an abstraction in
the `Core` project:

```csharp
public interface IWidgetDataAccess
{
    int Create(Widget widget);

    Widget GetWidget(int id);

    IList<Widget> GetWidgets(Expression<Func<Widget, bool>> filter);
}
```

This makes sense when using the 'clean architecture' patterns. If you are using the 'N-tier' patterns,
it may make more sense to include these abstractions in the `Infrastructure` project. In small
projects this can work quite well, as long as the components that implement the abstractions are
`internal` not `public` classes.

You could also consider creating a separate Class Library for these abstractions (e.g. `MyApp.Abstractions`):

```text
MyApp.sln
=> MyApp.Api
=> MyApp.Core
=> MyApp.Abstractions
=> MyApp.Infrastructure
    => Data
    => ThirdPartyApi
```

The important idea is that the `Core` project should only depend on these abstractions, not the
implementations. This is the common thread between all these approaches.

### What is the Best Level of Abstraction for Infrastructure Components?

Introducing abstractions is usually done to make the code easier to understand and maintain.
Choosing the best level of abstraction can greatly depend on your context. It may be driven by the
experiences and personalities of your team. It could be driven by standards the team must adhere to.
Or it could be driven by components being abstracted. In the end, it depends on a number of factors
that you will need to consider.

Let's take the data access abstractions. Some will argue that using the Repository pattern is a good
abstraction for data access. Others will argue that the built-in abstractions of Entity
Framework are all you should require. Some might argue that the interface should only expose
low-level api (Create/Read/Update/Delete) methods. Others might prefer to include more complex
queries to be encapsulated behind a data access abstraction.

For example, the `IWidgetDataAccess` above might expose some additional methods:

```csharp
public interface IWidgetDataAccess
{
    int Create(Widget widget);

    int CreateOrUpdate(Widget widget);

    Widget GetWidget(int id);

    Widget GetWidgetIncludingMetadata(int id);

    IList<Widget> GetWidgets(Expression<Func<Widget, bool>> filter);

    IList<Widget> GetInStockWidgets();
}
```

Regardless of which approach you prefer, it's the ability to use the abstractions within your core
application logic that remains.

Let's take the System Clock, as another example. If we need an abstraction for `DateTimeOffset.UtcNow`,
should the abstraction be a mirror of the underlying implementation?

```csharp
public interface ISystemClock
{
    DateTimeOffset UtcNow
}
```

Or should there be a new abstraction?

```csharp
public interface ISystemClock
{
    Instance GetCurrentInstant()
}
```

The best advice is to follow fundamental design rules for abstraction and encapsulation. If you do, then
you will unlikely make a choice that you can't undo if you don't like it. I typically recommend that
you create abstractions that closely mimic the underlying implementation. At least start there. It
reduces the burden on you and your team to create new abstractions when you may not be clear exactly
how they will end up being used. Over time, you should be able to refactor the code if and when a
new abstraction makes more sense in your context.

### Dependency Injection Considerations

The only way any of this works is with the assistance of Dependency Injection. .NET applications
have a good dependency injection framework built-in, making it easy to inject implementations of
the abstractions. But you have some choices to make on where the rules of the dependency injection
framework are stored.

You might start with them being in the `Program.cs` file:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddScoped<ISystemClock, SystemClock>();
```

But you can move these to another project, like the Infrastructure project using extension methods:

```csharp
public static class ServiceRegistrations
{
    public static IServiceCollection AddSystemClock(IServiceCollection services)
    {
        services.AddScoped<ISystemClock, SystemClock>();

        return services;
    }
}
```

If you move these registrations to the `Infrastructure` project, you will need to add a reference to
the `Microsoft.Extensions.DependencyInjection.Abstractions` NuGet package:

```powershell
dotnet add package Microsoft.Extensions.DependencyInjection.Abstractions
```

### Summary

Infrastructure concerns are one of the most important influences on how you design and organize your
code. Where possible, use abstractions for all external components to allow your code to be easy to
test, easy to understand and easy to evolve and grow.
