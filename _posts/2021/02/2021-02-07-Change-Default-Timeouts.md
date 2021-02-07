---
layout: post
title: 'Changing Default Time-outs for ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Sometimes you may need to change the default time-outs. I often have to search numerous articles to find the answers. I hope in the future I'll know to come here for all the details.

<!--more-->

### HTTP Requests in IIS

Prior to ASP.NET Core, the default request time-out in IIS is 110 seconds for .NET Framework applications. It can be modified in the Web.config file for the application via the `httpRuntime` section:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.web>
    <httpRuntime executionTimeout="110" />
  </system.web>
</configuration>
```

Reference: [https://docs.microsoft.com/en-us/dotnet/api/system.web.configuration.httpruntimesection.executiontimeout?view=netframework-4.8](https://docs.microsoft.com/en-us/dotnet/api/system.web.configuration.httpruntimesection.executiontimeout?view=netframework-4.8)

Note that this time-out applies only if the debug attribute in the `<compilation>` element is set to `false`.

However, for an ASP.NET Core application, the time-out is controlled with the `aspNetCore` section of the Web.config file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <aspNetCore requestTimeout="00:02:00" ...../>
  </system.webServer>
</configuration>
```

The default is configured with a timeout of 2 minutes (120 seconds).

Reference: [https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/aspnet-core-module?view=aspnetcore-2.1](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/aspnet-core-module?view=aspnetcore-2.1)

### Entity Framework Core Queries/Commands

A long-running command is typically encountered when reading data, rather than manipulating it. The default time-out for an EF Core command is 30 seconds. But, when calling `SaveChanges`/`SaveChangeAsync`, EF Core will execute multiple commands to persist all the changes. It will attempt to batch the changes to minimize the number of commands, rather than executing a separate command for each change. Regardless, each command is given a separate time-out of 30 seconds.

To change the timeout, use the `CommandTimeout` method via the options builder:

```csharp
    const int commandTimeoutInSeconds = 30;
    
    var contextOptions = new DbContextOptionsBuilder<SmithContext>();
    contextOptions
        .UseSqlServer(configuration["Database:ConnectionString"], options => {
          options.CommandTimeout(commandTimeoutInSeconds)
        });
```

Reference: [https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.infrastructure.sqlserverdbcontextoptionsbuilder?view=efcore-5.0](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.infrastructure.sqlserverdbcontextoptionsbuilder?view=efcore-5.0)

All queries (including uses of `FromSql`, `FromSqlInterpolated` or `FromSqlRaw`) should use this time-out value.

#### Explicit Commands

Sometime you may wish to execute an explicit command:

```csharp
using (var command = Database.GetDbConnection().CreateCommand())
{
  ...
}
```

These commands do not use the configured time-out and should be set manually: 

```csharp
using (var command = Database.GetDbConnection().CreateCommand())
{
  command.CommandTimeout = context.Database.GetCommandTimeout() ?? 30; 
  ...
}
```

#### TransactionScope

The default time-out for TransactionScope is 10 minutes. This is set at the machine-level. It can be adjusted when constructing:

```csharp
var scope = new TransactionScope(
    TransactionScopeOption.Required,
    new TransactionOptions {IsolationLevel = level, Timeout = TimeSpan.FromMinutes(5)},
    TransactionScopeAsyncFlowOption.Enabled);
```

### HttpClient Requests 

The default timeout when using HttpClient is 100 seconds. It can be adjusted when registering the instance in the `Startup` class:

```csharp
services.AddHttpClient("myClient", client =>
{
    client.Timeout = TimeSpan.FromSeconds(180);
});
```
