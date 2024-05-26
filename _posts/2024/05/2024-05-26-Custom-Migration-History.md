---
layout: post
title: 'Custom EF Core Migration History'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I want to know a bit detail of when data migrations are applied to my database using Entity Framework Core migrations.

<!--more-->

I am always looking at ways of getting better telemetry out of my .NET applications. One area that is does not provide a lot of detail is when data migrations are being applied. In our .NET applications we have a couple of different ways to apply data migrations. In some cases we run the migrations prior to the application starting. In other cases we run the migrations whenever the health check endpoint it requested. But we don't log that any migrations were pending, and we don't log the date/time when they were applied to the system. This often comes up when diagnosing odd behavior in production. 

### Custom Migration History

Each data migration applied to a SQL Server database is recorded in a table called `__EFMigrationsHistory`. The table has 2 columns: 
- `MigrationId` - the name of the migration
- `ProductVersion` - the EF Core version being used

I'd like to add a column that stores the date/time when the migration was applied (`MigratedAt`).

There is a class called `HistoryRepository` you can customize to modify the table being built. The problem is that this is a generic class, not specific to SQL Server. However, there is a `SqlServerHistoryRepository` that inherits from the `HistoryRepository`, and this makes it straightforward:

```csharp
public class CustomSqlServerHistoryRepository : SqlServerHistoryRepository
{
    public CustomSqlServerHistoryRepository(HistoryRepositoryDependencies dependencies) : base(dependencies)
    {
    }

    protected override void ConfigureTable(EntityTypeBuilder<HistoryRow> history)
    {
        base.ConfigureTable(history);

        history.Property<DateTime>("MigratedAt")
            .HasColumnType("datetime2")
            .HasDefaultValueSql("GETUTCDATE()")
            .IsRequired();
    }
}
```

Note that the `SqlServerHistoryRepository` is an internal class. The warning in the code is:

> *This is an internal API that supports the Entity Framework Core infrastructure and not subject to the same compatibility standards as public APIs. It may be changed or removed without notice in any release. You should only use it directly in your code with extreme caution and knowing that doing so can result in application failures when updating to a new Entity Framework Core release.*

All good advice. But this internal class has always been part of EF Core's codebase. Since it hasn't been removed in 6+ years, I think it is safe to reference. If it ever does get removed, we can deal with it then.

To use this custom class, you configure it as part of the context using the `ReplaceService` method:

```csharp
internal class AccountingDbContextDesignTimeDbContextFactory : IDesignTimeDbContextFactory<AccountingDbContext>
{
    public AccountingDbContext CreateDbContext(string[] args)
    {
        var configuration = new ConfigurationBuilder()
            .SetBasePath(Path.GetDirectoryName(Assembly.GetExecutingAssembly()?.Location) ?? ".")
            .AddJsonFile("appsettings.json")
            .Build();

        return new AccountingDbContext(CreateOptions(configuration));
    }

    public static DbContextOptions<AccountingDbContext> CreateOptions(IConfiguration configuration)
    {
        var contextOptions = new DbContextOptionsBuilder<AccountingDbContext>();

        contextOptions.UseSqlServer(configuration["Database:AmbitionAccounting"]);
        contextOptions.ReplaceService<IHistoryRepository, CustomSqlServerHistoryRepository>();

        return contextOptions.Options;
    }
}
```

### Custom Logging

If you run the migrations from the command line, you can see which migrations are getting applied. You can however run the migrations from within your code:

```csharp
var host = builder.Build();

using (var scope = host.Services.CreateScope())
{
    using var context = scope.ServiceProvider.GetRequiredService<AccountingDbContext>();
    
    context.Database.Migrate();
}

host.Run();
```

Unfortunately there are no logs captured by this process. So you don't know if any migrations were applied. But the context.Database object can provide you with this data. It has three methods to let you know what migrations are included in the code, and whether they have been applied or not: 

```csharp
var migrations = context.Database.GetMigrations();

var applied = context.Database.GetAppliedMigrations();

var pending = context.Database.GetPendingMigrations();
```

With this information, you can now provide some additional logging:

```csharp
using (var scope = host.Services.CreateScope())
{
    var logger = scope.ServiceProvider.GetRequiredService<ILogger<Program>>();

    using var context = scope.ServiceProvider.GetRequiredService<AccountingDbContext>();

    var pending = context.Database.GetPendingMigrations();
    pending.ToList()
        .ForEach(m => logger.LogInformation("Pending migration: {MigrationName}", m));

    if (pending.Any())
    {
        logger.LogInformation("Applying migrations");

        context.Database.Migrate();
    }
}
```

### Summary

It is possible to include additional columns in the `__EFMigrationsHistory` table. I've shown how to add the date/time stamp when the migration was applied, but you could add more if desired (application version for example). I've also shown how you can log when there are pending migrations to include these details in your application's telemetry.
