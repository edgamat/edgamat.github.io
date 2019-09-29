---
layout: post
title: 'Using ASP.NET Core in a Clean Architecture'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Having Domain business logic in an application is the first step in building with a Clean Architecture. Now
let's take a look at incorporating the rest of the application layers. We'll start with the addition of the ASP.NET
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
