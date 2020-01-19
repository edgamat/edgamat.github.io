---
layout: post
title: 'Custom Serilog Configurations with ASP.NET Core 3'
author: 'Matthew Edgar'
published: '2020-01-26'
excerpt_separator: <!--more-->
---

I am particular about the logs my application generate. Here are some ways I have found
to customize Serilog with ASPNET Core 3.
<!--more-->

As I have previously written about, Serilog comes with a dedicated package for ASP.NET Core support.
I prefer to store the configuration information in the `appsettings.json` and to send the logging 
output to the console. 

Here are the packages you will need to install:

```powershell
dotnet add package Serilog
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Settings.Configuration
dotnet add package Serilog.Enrichers.AssemblyName
```

First thing is to remove the "Logging" node from the `appsettings.json` as Serilog makes no use of them.

Then, add the following:

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "System": "Warning",
        "Microsoft": "Warning"
      }
    }
  }
}
```

This let's Serilog log Information level messages and higher only. This is pretty much the way you want things in all environments.

## Console

```powershell
dotnet add package Serilog.Sinks.Console
```

Logging to the Console is a necessity for local development. Here is my usual configuration:

```json
{
  "Serilog": {
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "theme": "Serilog.Sinks.SystemConsole.Themes.AnsiConsoleTheme::Code, Serilog.Sinks.Console",
          "outputTemplate": "[{Timestamp:HH:mm:ss.fff K} {Level:u3}] {Message:lj} <s:{SourceContext}>{NewLine}{Exception}"
        }
      }
    ]
  }
}
```

## File

```powershell
dotnet add package Serilog.Sinks.File
```

When using a file to store logs, here is a typical configuration:

```json
{
  "Serilog": {
    "WriteTo": [
      { 
        "Name": "File",
        "Args": {
          "path": "info.log",
          "rollingInterval": "Day",
          "outputTemplate": "[{Timestamp:HH:mm:ss.fff K} {Level:u3}] [{AssemblyVersion}] {Message:lj} <s:{SourceContext}>{NewLine}{Exception}"
        }
      }
    ]
  }
}
```

## SQL Server

```powershell
dotnet add package Serilog.Sinks.MSSqlServer
```

NOTE: Use 5.1.3 or higher. Version 5.1.2 of the NuGet package does not allow for customization of the table.

```json
{
  "Serilog": {
    "WriteTo": [
      {
        "Name": "MSSqlServer",
        "Args": {
          "connectionString": "Server=localhost;Database=MyDatabase;Integrated Security=true;",
          "tableName": "SampleLogs10",
          "autoCreateSqlTable": true,
          "restrictedToMinimumLevel": "Information",
          "columnOptionsSection": {
            "removeStandardColumns": [ "Properties" ],
            "additionalColumns": [
              { "columnName": "AssemblyVersion", "dataType": "VarChar", "dataLength": 32 },
              { "columnName": "RequestId", "dataType": "VarChar", "dataLength": 32 }
            ],
            "message": { "columnName": "MessageValue" },
            "level": { "columnName": "Severity", "storeAsEnum": false, "dataLength": 32 },
            "timeStamp": { "columnName": "LoggedAt", "convertToUtc": true, "dataType": "DateTime2" }
          }
        }
      }
    ],
    "Enrich": [ "FromLogContext", "WithAssemblyVersion" ]
  }
}
```



