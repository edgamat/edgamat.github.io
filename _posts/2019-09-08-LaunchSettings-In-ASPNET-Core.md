---
layout: post
title: 'Launch Settings with ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Here are some notes on how to override the `appsettings.json` using `launchSettings.json` in the Properties folder to control ASP.NET Core applications on your local workstation.

<!--more-->

There are several good articles to provide an overview of the `launchSettings.json`:

- [How to set the hosting environment in ASP.NET Core](https://andrewlock.net/how-to-set-the-hosting-environment-in-asp-net-core/)

- [What is launchSettings.json in ASP.NET Core](https://www.talkingdotnet.com/launchsetting-json-in-aspnet-5/)

- [Using .NET Core launchSettings.json to run/debug apps in Rider](https://blog.jetbrains.com/dotnet/2018/11/08/using-net-core-launchsettings-json-rundebug-apps-rider/)

- [Use multiple environments in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments)

To quote the .NET Core documents:

> The environment for local machine development can be set in the `Properties\launchSettings.json` file of the project. Environment values set in `launchSettings.json` override values set in the system environment.

The ASP.NET Core framework contains a built-in mechanism for configuring the application. I suspect it was born out of the
need to make it easier to configure web application hosted in Azure. Azure uses environment variables to inject settings into
the websites it hosts. This makes sense since environment variables are pretty universal, and work well with Windows and Linux
hosted applications. The built-in mechanism relies on the simple notion that all application settings can be reduced down
to a set of key value pairs.

Microsoft incorporates a few configuration providers as part of the ASP.NET Core web application framework. By default,
the `WebHost.CreateDefaultBuilder` routine (called in the `Program` class), includes settings from the following providers:

- `Microsoft.Extensions.Configuration.Json`
- `Microsoft.Extensions.Configuration.UserSecrets`
- `Microsoft.Extensions.Configuration.EnvironmentVariables`
- `Microsoft.Extensions.Configuration.CommandLine`

Note that order matters. After loading the settings from the JSON files, then it loads the user secrets, followed by
the environment variables and lastly the command line arguments. So if you have a settings, say `"Security:SessionTimeout"`
defined in your JSON File:

```json
{
  "Security": {
    "SessionTimeout": 20
  }
}
```

And you have an environment variable with the same name:

`"Security:SessionTimeout" = 25`

The configuration value loaded into the application will be 25, not 20. The same override can be made via the command line.

This is all laid out in more detail here: [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/)

## Using the Launch Settings

So in the `launchSettings.json` file, find the profile you want to override. In my case, I usually use the Project profile,
names after the Project being run (usually the last one in the file). Add an `environmentVariables` node:

```json
{
  "profiles": {
    "RefArch01": {
      "commandName": "Project",
      "launchBrowser": true,
      "launchUrl": "api/values",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Local"
      },
      "applicationUrl": "http://localhost:51715/"
    }
  }
}
```

In this example, I am overriding the `ASPNETCORE_ENVIRONMENT` variable to use a `Local` environment configuration, rather
than the ones that Microsoft configures automatically. The `WebHost.CreateDefaultBuilder` automatically includes the
settings from the `appsettings.{Environment}.json` file, which is not always desirable. Setting it to `Local` means
only the `appsettings.json` file is processed, ignoring the `Development` settings, for example.

To override the SessionTimeout value of the Security node, add this variable:

```json
"environmentVariables": {
  "ASPNETCORE_ENVIRONMENT": "Local",
  "Security:SessionTimeout": 25
},
```

## How to handle arrays

When configuring Serilog, the `appsettings.json` file contains an array of Sinks to log messages. Overriding that array is
possible as long as you get the syntax correct.

Let's say you have the following:

```json
{
  "PossibleValues": [{ "Name": "Value1" }, { "Name": "Value2" }, { "Name": "Value3" }]
}
```

This results in the following values being added to the configuration:

`PossibleValues:0:Name`  
`PossibleValues:1:Name`  
`PossibleValues:2:Name`

This is easy enough to override via environment variables:

`PossibleValues:1:Name = "Value42"`

You can even add elements to the array:

`PossibleValues:3:Name = "Value42"`

However, if you want to reduce the size of the array, then you are out of luck. Environment variables cannot be set
to an empty string (or anything equivalent). So if you need to adjust the array, you either have to do it
using a provider other than the environment variables, or you have to adjust the array in code (i.e programmatically).
