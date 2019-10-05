---
layout: post
title: 'Using ASP.NET Core in a Clean Architecture'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Having Domain business logic in an application is the first step in building with a Clean Architecture. Now
let's take a look at incorporating the application and UI layers. Specifically, the addition of the ASP.NET
Core framework.

<!--more-->

In the previous post we established the Domain business logic using a .NET Standard Class Library. We were careful
to not introduce any unnecessary dependencies. As we introduce the ASP.NET Core framework we need to ensure to
keep this independence intact.

## Adding the ASP.NET Core Framework

We are building a REST API using ASP.NET Core. Adding a new ASP.NET Core project to our solution can be done using
the following script:

```powershell
$projectName = "BookWarehouse"

md "$projectName\src\$projectName.Api"

cd "$projectName\src\$projectName.Api"
dotnet new webapi --no-restore
dotnet add reference "..\$projectName.Domain\$projectName.Domain.csproj"

cd ..\..\
dotnet sln add ".\src\$projectName.Api\$projectName.Api.csproj"

cd ..
```

## So Far, So Good

Let's assume the Warehouse API allows callers to retrieve all Bookshelf entities. We would expose
an GET endpoint similar to this:

```csharp
public class WarehouseController : ControllerBase
{
    private readonly Warehouse _warehouse;

    public WarehouseController(Warehouse warehouse)
    {
        _warehouse = warehouse;
    }

    [HttpGet]
    public ActionResult<IEnumerable<BookShelf>> Get()
    {
        return Ok(_warehouse.GetBookShelves());
    }
}
```

This usage does not require the Domain to know anything about the application layer (i.e. the ASP.NET Core framework).
To maintain this relationship we need to ensure that on business rules are not implemented in the ASP.NET project
and that no user interface processing logic exists in the Domain project.

## Wiring Up The Dependency Injection Container

ASP.NET Core has a built-in Dependency Injection Container which allows applications to compose objects are
run-time. This permits the dependencies to be injected (typically through constructor parameters) into
objects, rather than having them be aware of the implementation of these dependencies.

For example, if the code requires the `WarehouseController` to create its instance of the `Warehouse` object,
we would introduce a new dependency that controller would need to manage. This is a poor design choice since the
way a `Warehouse` object is created could depend on a connection to a database or configuration settings and now
the controller would need to be aware of all these things. It is wiser to let the Dependency Injection Container
to construct the `Warehouse` instance and pass it to the controller.

In the `Startup` class, there is a method where you can add this logic:

```csharp
void ConfigureServices(IServiceCollection services)
```

To add the `Warehouse` construction logic to the application, simply add the following:

```csharp
services.AddScoped<Warehouse>();
```

This will create a new instance of the `Warehouse` for each request made to the `WarehouseController`. It will
use the default parameter-less constructor to create the `Warehouse` instance.

As the application evolves and database persistance is added, the logic to create the `Warehouse` is managed
(centrally) in the DI Container, and not in the controllers that use the objects.

## Where to Next

Adding the ASP.NET Core application is straightforward and doesn't require the Domain business logic to be
aware (dependent) on this new layer to the application. But what about persisting the data to a database?
That will be the topic of our next port and will begin an interesting adventure with Entity Framework Core.
