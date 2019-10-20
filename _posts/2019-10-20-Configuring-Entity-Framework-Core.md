---
layout: post
title: 'Configuring Entity Framework Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Entity Framework (EF) Core, provides a dizzying array of configuration options. In a Clean Architecture,
configuring EF Core is straightforward. Let's explore how.

<!--more-->

## The Approach

The goal here will be to configure EF Core to use SQL Server for persistence in a Clean Architecture project.

Recall how this was presented in the previous post:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var options = CreateOptions(Configuration);
    services.AddScoped(p => new WarehouseContext(options));

    services.AddScoped<IBookRepository, BookRepository>();
}
```

Here's an example of what the `CreateOptions` routine accomplishes:

```csharp
private DbContextOptions<WarehouseContext> CreateOptions(IConfiguration configuration)
{
    var contextOptions = new DbContextOptionsBuilder<WarehouseContext>();

    contextOptions.UseSqlServer(configuration["Database:ConnectionString"]);

    return contextOptions.Options;
}
```

In this example, SQL Server is being configured to work with EF Core. Regardless of the
database being used (SqlLite, Oracle, MySQL) they can all be configured in a similar fashion.

Prior to EF Core 3, you may also want to include the following:

```csharp
    var contextOptions = new DbContextOptionsBuilder<SmithContext>();
    contextOptions
        .UseSqlServer(configuration["Database:ConnectionString"])
        .ConfigureWarnings(warnings => warnings.Throw(RelationalEventId.QueryClientEvaluationWarning));
```

Adding this warning will alert you to cases where EF Core cannot convert a Linq expression into
a SQL query. Without this warning configured to throw an exception, you may experience performance
issues because EF Core will only perform part of the filtering using SQL. Here's a post that goes into
more detail:

[https://gist.github.com/edgamat/5ed3918e725d35913720667d4237fbeb](https://gist.github.com/edgamat/5ed3918e725d35913720667d4237fbeb)

Thankfully, with EF Core, this is no longer an option as the ability to include client-evaluated functions
in a Linq expression will always throw an exception:

[https://docs.microsoft.com/en-us/ef/core/what-is-new/ef-core-3.0/breaking-changes#linq-queries-are-no-longer-evaluated-on-the-client](https://docs.microsoft.com/en-us/ef/core/what-is-new/ef-core-3.0/breaking-changes#linq-queries-are-no-longer-evaluated-on-the-client)

## EF Core Migrations

When using EF Core Migrations, you will need to create a design-time factory class that EF Core will
use to configure itself when performing migrations.

```csharp
using System;
using System.IO;
using System.Reflection;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Design;
using Microsoft.Extensions.Configuration;

namespace BookWarehouse.Persistence
{
    public class WarehouseContextFactory : IDesignTimeDbContextFactory<WarehouseContext>
    {
        public WarehouseContext CreateDbContext(string[] args)
        {
            var runtimePath = GetRuntimePath();

            var configuration = new ConfigurationBuilder()
                .SetBasePath(runtimePath)
                .AddJsonFile("appsettings.json")
                .Build();

            return new WarehouseContext(CreateOptions(configuration));
        }

        private static string GetRuntimePath()
        {
            var executingAssembly = Assembly.GetExecutingAssembly();
            var assemblyFile = executingAssembly.Location;

            return Path.GetDirectoryName(assemblyFile);
        }

        public static DbContextOptions<WarehouseContext> CreateOptions(IConfiguration configuration)
        {
            if (configuration == null)
                throw new ArgumentNullException(nameof(configuration));

            var contextOptions = new DbContextOptionsBuilder<WarehouseContext>();

            contextOptions.UseSqlServer(configuration["Database:ConnectionString"]);

            return contextOptions.Options;
        }
    }
}
```

The code uses the same `appsettings.json` as the `Api` project. First, add a link to the existing
file in the `BookWarehouse.Persistence.csproj` file:

```xml
  <ItemGroup>
    <Content Include="..\BookWarehouse.Api\appsettings.json" Link="appsettings.json" />
  </ItemGroup>
```

Then add two new NuGet package references to the `BookWarehouse.Persistence` project:

```
Microsoft.Extensions.Configuration.FileExtensions
Microsoft.Extensions.Configuration.Json
```

Now the new design-time factory class will compile.

To generate a new Migration, open a command prompt (or Powershell) in the root folder of the
persistence project:

```
C:\code\BookWarehouse\src\BookWarehouse.Persistence
```

Run the Entity Framework command for migration creation `dotnet ef migrations add <migration name>`

## Next Steps

So far so good. As your project grows you may find that storing all the model binding details in the
single `WarehouseContext` file difficult to manage. Next time we'll take a look at some techniques
to help manage that and more.
