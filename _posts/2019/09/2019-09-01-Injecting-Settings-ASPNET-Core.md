---
layout: post
title: 'Injecting Configuration Settings in ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

ASP.NET Core has a very robust Dependency Injection (DI) framework. Configuring the application is
always necessary at some level. Most applications have a need to inject configuration settings into
the DI container for other classes to use. Let me show one way that has worked well for me.

<!--more-->

In this example, a class is responsible for returning a set of values. The
number of values returned by the class is controlled via a configuration setting:

```csharp
public class ValueSettings
{
    public int ValuesToReturn { get; set; }
}

public class ValueService
{
    private readonly ValueSettings _settings;

    public ValueService(ValueSettings settings)
    {
        _settings = settings;
    }
}
```

To inject the `ValueSettings`, first you need to load the settings from the configuration. Here is one way:

```csharp
services.Configure<ValueSettings>(configuration.GetSection(nameof(ValueSettings)));
services.AddSingleton(e => e.GetService<IOptions<ValueSettings>>().Value);
```

NOTE: `configuration` is the instance of `IConfiguration` injected into the `Startup` class by the ASP.NET Core hosting environment.

The first statement uses the built-in `IOptions` framework that implements the [Options pattern][op]{:target="\_blank"}.
It injects an instance of `IOptions<ValueSettings>` into the DI Container.

The `ValueService` requires the DI Container to provide `ValueSettings` rather than `IOptions<ValueSettings>`. The
second statement creates an additional entry in the DI container for `ValueSettings`.

One might wonder why the `ValueService` doesn't accept `IOptions<ValueSettings>` from the DI Container? Well, there are
a couple of reasons. First, it may not be possible to modify the source code that defines `ValueService`. It may be contained in
an assembly outside of your control. Second, accepting `IOptions<ValueSettings>` from the DI Container requires your service
class to accept an additional dependency (on `IOptions`). This is what I'd call a 'framework dependency' which is not something you typically want to include in your core domain model. Regardless of your reasons, injecting the `ValueSettings` can be a desirable design.

If you also want to avoid a dependency of the DI container on the `IOptions` here is an alternate way to load the settings:

```csharp
var settings = new ValueSettings();
configuration.Bind(nameof(ValueSettings), settings);
services.AddSingleton(settings);
```

I prefer to create an extension method to help make loading configuration more readable:

```csharp
public static void ConfigureSettings<TSettings>(this IServiceCollection services, IConfiguration configuration)
    where TSettings : class, new()
{
    var settings = BindSection<TSettings>(configuration, typeof(TSettings).Name);
    services.AddSingleton(settings);
}

public static TConfig BindSection<TConfig>(this IConfiguration configuration, string section) where TConfig : class, new()
{
    var config = new TConfig();
    configuration.Bind(section, config);

    return config;
}

...

services.ConfigureSettings<ValueSettings>(Configuration);
```

[op]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options
