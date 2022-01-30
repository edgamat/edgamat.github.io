---
layout: post
title: 'Detecting Captive Dependencies in ASP.NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

If you aren't careful, you can extend the lifetime of a service using Dependency Injection with ASP.NET. Let's see how to detect Captive Dependencies. 

<!--more-->

### What are Captive Dependencies?

.NET Core implements three lifetime styles for objects registered with the DI Container:

| Name      | Description                                                                                                        |
| --------- | ------------------------------------------------------------------------------------------------------------------ |
| Transient | Objects are created each time they're requested                                                                    |
| Singleton | Objects are created the first time they're requested and a single instance exists until the application shuts down |
| Scoped    | Objects are created once per scope (e.g. in ASP.NET this is once per request)                                      |

It is possible to inject a scoped service into a service registered as a singleton. Because the singleton service doesn't get disposed of until the application shuts down, the scoped service inherits this same lifetime. It doesn't matter that the service is registered with a scoped lifetime. It depends on how it is used in the code. 

This is [something Mark Seemann calls a captive dependency](https://blog.ploeh.dk/2014/06/02/captive-dependency/). Mark is the co-author of the fantastic book "Dependency Injection Principles, Practices, and Patterns". He describes a captive dependency occurring when a mis-configured lifetime leads to a longer-lived service holding a shorter-lived service _**captive**_. 

A year ago I wrote about a memory leak that was the result of a captive dependency:

[Memory Leaks using Dependency Injection with .NET Core]({% post_url 2021/01/2021-01-02-Memory-Leaks-Using-DI-With-NET-Core %})

Detecting the captive dependency due to the mis-use of the shorter-lived service within a singleton took a lot of time and effort. It would be preferable to detect these mis-configurations earlier in the development/use of the application.

### ValidateScopes and ValidateOnBuild

The default service provider includes two options to detect one of the most likely cases: consuming a scoped service from a singleton service.

The `ValidateScopes` option verifies that scoped services never gets resolved from root provider (which is what occurs when consuming a scoped service from a singleton service).

The `ValidateOnBuild` option verifies that all services can be created during the call to `BuildServiceProvider()`.

Both these properties can be configured in the `Program` of the default ASP.NET application template:


```csharp
public static void Main(string[] args)
{
    CreateHostBuilder(args).Build().Run();
}

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        })
        .UseDefaultServiceProvider((context, options) => {
            options.ValidateScopes = true;
            options.ValidateOnBuild = true;
        });
```

Using `ValidateScopes` by itself will result in an exception being thrown when a scoped service is resolved from the root provider. This occurs when the application resolves a singleton service with a captive scoped dependency.

Using `ValidateOnBuild` will raise an exception when the service provider is built, which occurs on startup. Note, you must set `ValidateScopes` to `true` when setting `ValidateOnBuild` to `true` or else the captive dependency won't be detected.

This is a step in the right direction. I would recommend these be turned on by default in the development environment.

### What about Transient Lifetime Services?

The `ValidateScopes` option does not detect captive dependencies when a singleton service consumes a transient lifetime service. This makes sense, in-so-much that there could be some instances where consuming a transient service is not a problem. One assumes that it is never acceptable for a scoped service to be consumed by a singleton, but there are scenarios where consuming a transient service in a singleton has no bad side-effects.

However, it is still a captive dependency and it would be preferable to be able to detect these somehow.

### Can We Do Better?

It would be preferable if captive dependencies could be detected earlier in the software development process. Mark Seemann has written extensively on his preference of foregoing the use of a DI Container with a preference for [Pure DI](https://blog.ploeh.dk/2014/06/10/pure-di). His advice is sound; don't use a DI container if your context would be better served using a simple alternative. Using Pure DI (which hand-codes the dependency graph) does come with some higher maintenance costs. But in some situations it may be more explicit and capable of giving you better feedback at design/compile time. It is not as likely that a captive dependency would go undetected using Pure DI. 

But I'm going to assume that a lot of development teams use the built-in dependency injection container for ASP.NET Core. This has certainly been my experience. In this context, I want to look for ways to give myself more feedback, earlier in the process. One of the biggest issues with detecting DI container issues is how subtle the problems can be when you look at the code.

I can think of two possible improvements, using unit tests. My team has an automated CI/CD pipeline that ensures all unit tests must pass before letting the code be deployed to a target environment. So I can be relatively sure that if a I write a unit test that runs some assertions on the configuration of the DI Container, it will detected issues prior to the code being merged into the main code branch.

The first unit test I can imagine is querying the registered services and looking for captive dependencies. If any are found then the unit test would fail. The trouble with this sort of unit test is its complexity and relative benefits. The only thing it would really give me above what the 'Validate Scopes' features provide is the hope of detecting situations where a transient service is injected into a scoped service. But this is not always a bad thing, so you'd have to include some ability to ignore some of these configurations. 

I think a better alternative would be a simpler unit test that attempts to build the configured container using the `ValidateScopes` and `ValidateOnBuild` options. It should detect the invalid configuration of a scoped service being consumed by a singleton service.

### Summary

Captive dependencies are a dependency injection anti-pattern that can have some subtle or impactful side effects in you application. Detecting these mis-configurations of your ASP.NET DI Container can be made easier through the use of the `ValidateScopes` and `ValidateOnBuild` options when building the DI Container. There is little risk of having these enabled in your development environment as they will give you advance warning of something configured incorrectly.

But captive dependencies are sometimes subtle, especially when using transient services. In most situations, knowledge of dependency injection concepts and patterns is more effective in preventing captive dependencies. Help your team understand the issues and what to look for in the code.



