---
layout: post
title: 'Windows Event Logs with Serilog'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

If your .NET Core application needs to store log entries in a Windows Event Log, Serilog has a great option. However, the samples and API documentation aren't that great. So here are my notes on my experiences.

<!--more-->

### Installation

The first thing you will need to do is download and install the necessary Serilog Nuget packages:

```powershell
dotnet add package Serilog
dotnet add package Serilog.Sinks.EventLog
```

Then you'll need to configure the sink in your code:

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.EventLog(
        source: "MyTestSource",
        logName: null,
        machineName: ".",
        manageEventSource: false,
        restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Verbose,
        outputTemplate: "{Message}{NewLine}{Exception}")
    .CreateLogger();
```

Using a Powershell window with Administrative privileges, add the source to the event log:

```powershell
New-EventLog -LogName "Application" -Source "MyTestSource"
```

### Options

Let's run through the options:

`source: "MyTestSource"`  
The source name by which the application is registered on the local computer. There is no default for this parameter and it is a required value.

`logName: null`  
The name of the log the source's entries are written to. Possible values include Application, System, or a custom event log. The default is to use the Application log.

`machineName: "."`  
The name of the machine hosting the event log written to. The local machine by default.

`manageEventSource: false`
If true, check/create event source as required. Defaults to false i.e. do not allow sink to manage event source creation.

`restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Verbose`
The minimum log event level required in order to write an event to the sink. To prevent un-necessary entries, I usually restrict this to `Serilog.Events.LogEventLevel.Warning`.

`outputTemplate: "{Message}{NewLine}{Exception}")`
A message template describing the format used to write to the sink.

### Configuration

If you wish to configure the Serilog Event Log via the .NET Core Configuration extensions, you can do so as well:

```powershell
dotnet add package Serilog.Settings.Configuration
dotnet add package Microsoft.Extensions.Configuration.Json
```

```csharp
var builder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json");

var configuration = builder.Build();

Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(configuration)
    .CreateLogger();
```

Here's what the `appsettings.json` looks like:

```json
{
    "Serilog": {
        "WriteTo": [
            {
                "Name": "EventLog",
                "Args": {
                    "source": "SerilogEventLogNotes",
                    "restrictedToMinimumLevel": "Warning"
                }
            }
        ]
    }
}
```

### Controlling EventId Values

In some scenarios you may wish to provide custom EventId values for your log entries. The library includes a `IEventIdProvider` interface you can implement your own custom class for:

```csharp
/// <summary>
/// Event Id provider for log events
/// </summary>
public interface IEventIdProvider
{
    /// <summary>
    /// Computes an Event Id for the given log event.
    /// </summary>
    /// <param name="logEvent">The log event to compute the event id from.</param>
    /// <returns>Computed event id based off the given log.</returns>
    ushort ComputeEventId(LogEvent logEvent);
}
```

One scenario that comes up is to assign specific EventId values to exceptions (or groups of exceptions). Here I have created a simple implementation to demonstrate the idea:

```csharp
using System;
using Serilog.Events;
using Serilog.Sinks.EventLog;

namespace SerilogEventLogNotes
{
    public class CustomEventIdProvider : IEventIdProvider
    {
        public ushort ComputeEventId(LogEvent logEvent)
        {
            if (logEvent?.Exception == null)
            {
                return (ushort)Compute(logEvent.MessageTemplate.Text);
            }

            switch (logEvent.Exception)
            {
                case NullReferenceException _:
                    return (ushort)1000;

                case FormatException _:
                    return (ushort)2000;

                default:
                    return (ushort)9999;
            }
        }

        /// <summary>
        /// Compute a 32-bit hash of the provided <paramref name="messageTemplate"/>. The
        /// resulting hash value can be uses as an event id in lieu of transmitting the
        /// full template string.
        /// </summary>
        /// <param name="messageTemplate">A message template.</param>
        /// <returns>A 32-bit hash of the template.</returns>
        static int Compute(string messageTemplate)
        {
            if (messageTemplate == null) throw new ArgumentNullException(nameof(messageTemplate));

            // Jenkins one-at-a-time https://en.wikipedia.org/wiki/Jenkins_hash_function
            unchecked
            {
                uint hash = 0;
                for (var i = 0; i < messageTemplate.Length; ++i)
                {
                    hash += messageTemplate[i];
                    hash += (hash << 10);
                    hash ^= (hash >> 6);
                }
                hash += (hash << 3);
                hash ^= (hash >> 11);
                hash += (hash << 15);

                //even though the api is type int, eventID must be between 0 and 65535
                //https://msdn.microsoft.com/en-us/library/d3159s0c(v=vs.110).aspx
                return (ushort)hash;
            }
        }
    }
}
```

**NOTE**: The `Compute` function is available from the GitHub source for the EventLog library:

https://github.com/serilog/serilog-sinks-eventlog/blob/dev/src/Serilog.Sinks.EventLog/Sinks/EventLog/EventIdHashProvider.cs

To use the new provider, here is the syntax:

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.EventLog(
        source: "SerilogEventLogNotes",
        restrictedToMinimumLevel: Serilog.Events.LogEventLevel.Warning,
        eventIdProvider: new CustomEventIdProvider())
    .CreateLogger();
```

Or via the `appsettings` configuration format:

```json
{
    "Serilog": {
        "WriteTo": [
            {
                "Name": "EventLog",
                "Args": {
                    "source": "SerilogEventLogNotes",
                    "restrictedToMinimumLevel": "Warning",
                    "eventIdProvider": "SerilogEventLogNotes.CustomEventIdProvider, SerilogEventLogNotes"
                }
            }
        ]
    }
}
```



