---
layout: post
title: 'Using Serilog with ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

[Serilog][serilog] is one of the more popular logging packages that integrates with ASP.NET Core. I have
recently used Serilog on several RESTFUL API projects and I have found it to work extremely well with
most logging needs.

<!--more-->

Serilog comes with a dedicated package for ASP.NET Core support. It makes it very easy to setup and
provides several options for where to send the logging information. My 'default' setup is to store
the configuration information in the `appsettings.json` and to send the logging output to the console. Once
this is set up properly, I can start adding more sinks (where the system places log messages).

Here are the packages you will need to install:

```powershell
dotnet add package Serilog
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Settings.Configuration
dotnet add package Serilog.Sinks.Console
```

Once installed, you'll need to configure Serilog. Place the following in your `appsettings.json`:

```json
"Serilog": {
  "MinimumLevel": {
    "Default": "Debug",
    "Override": {
      "Microsoft": "Warning"
    }
  },
  "WriteTo": [
    { "Name": "Console" }
  ],
  "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ]
}
```

This sets the minimum log level to `Debug` so we can see `Debug` or higher log messages in the sinks. The
`Override` on the `Microsoft` namespace restricts the logging in the ASP.NET Core pipeline to `Warning` level
messages or higher. I'd recommend you place around with these settings to find what is best for your needs.

Next, you'll need to initialize Serilog when the application starts. For that, we'll need to do some surgery to the
`Main` routine in the `Program.cs`. First change the return type from `void` to `int`. This allows the program to
return a non-zero value to the console if it exists with an error. this can be very useful in determining if
the application starts/ends successfully.

```csharp
public static int Main(string[] args)
```

Next, in the `Main` routine, we'll need to load the configuration data from the settings file. Here
is what I found works for me. It is by no means the best way so feel free to make adjustments here for
your own context.

My typical pattern is use the `appsettings.json` to store the settings used by developers on their local
workstations. I always provide a means to override these setting using environment variables. I then use
the `appsettings.{Environment}.json` file to override the settings when deployed to the target environment.

To get the target environment, you can use the following:

```csharp
var h = new WebHostBuilder();
var environment = h.GetSetting("environment"); // Development, Staging, Production, etc.
```

Now build a configuration instance and use it to configure the logger (using the `Log` static class).

```csharp
var configuration = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json")
    .AddJsonFile($"appsettings.{environment}.json", optional: true)
    .AddEnvironmentVariables()
    .Build();

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(configuration)
    .CreateLogger();
```

Next, let's add a `try/catch` block around the code that runs the web server. This allows the program
to log any issues that occur during startup.

```csharp
try
{
    Log.Information("Starting web host");

    CreateWebHostBuilder(args).Build().Run();

    return 0;
}
catch (Exception ex)
{
    Log.Fatal(ex, "Host terminated unexpectedly");
    return 1;
}
finally
{
    Log.Information("Ending web host");
    Log.CloseAndFlush();
}
```

And finally, we modify the `CreateWebHostBuilder` routine to use Serilog:

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseSerilog()
        .UseStartup<Startup>();
```

When you run your application, you should see something like:

```console
[05:50:13 INF] Starting web host
Hosting environment: Local
Content root path: C:\Users\edgamat\source\repos\RefArch
Now listening on: http://localhost:51715
Application started. Press Ctrl+C to shut down.
[05:50:15 WRN] Failed to determine the https port for redirect.
```

You can see that the first line contains the `INF` message we added in the `Main` routine.

Now you should be able to add loggers to you program and see the output in the Console window. I'll
create an additional blog post to demonstrate how that is accomplished.

### Using the SQL Server Sink

In case you want to use SQL Server to store your log messages, Serilog includes a package for that:

```powershell
dotnet add package Serilog.Sinks.MSSqlServer
```

And here is an example of how to configure it via the `appsettings.json` file:

```json
"WriteTo": [
  { "Name": "Console" },
  {
    "Name": "MSSqlServer",
    "Args": {
      "connectionString": "Server=localhost;Database=RefArch;Integrated Security=true",
      "schemaName": "dbo",
      "tableName": "ErrorLog",
      "autoCreateSqlTable": true,
      "restrictedToMinimumLevel": "Warning"
    }
]
```

[serilog]: https://github.com/serilog/serilog
