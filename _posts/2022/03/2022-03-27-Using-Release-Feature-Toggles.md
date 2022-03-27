---
layout: post
title: 'Using Release Feature Toggles for Trunk-Based Development'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When using trunk-based development, release feature toggles are a means to deploy your code without releasing it to users. Here are some ideas I have learned in using them.

<!--more-->

### What are Feature Toggles?

Martin Fowler has a great article introducing Feature Toggles:

**Feature Toggles (aka Feature Flags)**  
[https://www.martinfowler.com/articles/feature-toggles.html](https://www.martinfowler.com/articles/feature-toggles.html)

So summarize, feature toggles allow you to modify system behavior without modifying code. Whether a feature is available for use is controlled through a set of toggles (or flags) that are configured via application settings (like in a config/settings file) rather than embedded in the code.

In the context of a trunk-base development workflow, this enables teams to work on new features in the main/trunk branch of the codebase safely. They can disable the new functionality while it is being worked on and safely allow that code to be deployed into production. Once the feature is ready, it can be enabled via a configuration change (not a code change).

### Release Feature Toggles

While there are many types of feature toggles, the type I have used the most is Release Feature Toggles. Here is the description from Fowler's article:

> These are feature flags used to enable trunk-based development for teams practicing Continuous Delivery. They allow in-progress features to be checked into a shared integration branch (e.g. master or trunk) while still allowing that branch to be deployed to production at any time. Release Toggles allow incomplete and un-tested codepaths to be shipped to production as latent code which may never be turned on.

We try to ensure our codebase is always in a releasable state. At any moment, it should be possible to deploy the code currently in the trunk branch into production without any side effects. If I am working on a feature that would not allow us to do that, I need to use a release feature toggle to disable the work I am doing until it is ready for release.

Here is a typical example of how I use a release feature toggle. I am working on a new story to change how a calculation is performed. I expect to have the changes made in a single Sprint (we are following a Scrum agile process). We have a series of automated tests that run each time a new version of the code is deployed to our development or test environments. If I commit my new changes to the trunk branch, some of those tests will fail, which prevents the code from being deployed into production. 

### Embedding Toggle Points

There are a few ways to toggle your new feature on or off. Which technique you use depends on the context.

**Conditional Branching**  
This technique injects a toggle directly into a class. Methods in the class can use this toggle to control its internal behavior. This approach is best suited when the new feature has a small impact on the code.

```csharp
public class PaymentService
{
    private readonly bool _includeTaxesWithPayment;
    public PaymentService(PaymentConfiguration config)
    {
        _includeTaxesWithPayment = config.IncludeTaxesWithPayment;
    }
}
```

**Dependency Injection Branching**  
This technique uses a feature toggle to control via your application's Dependency Injection Container. Your feature has a separate implementation of a service and whether to use it or not is determined using a toggle within the Composite Root's configuration. 

```csharp
public IConfiguration Configuration { get; }

public void ConfigureServices(IServiceCollection services)
{
    if (Configuration.GetValue("UseNewShippingService"))
    {
        services.AddScoped<IShippingService, NewShippingService>();
    }
    else
    {
        services.AddScoped<IShippingService, OldShippingService>();
    }
}
```

This technique makes sense when the changes to the code are larger than just a single method in a class, or when you want to have a completely new set of unit tests for the new service, while keeping the remaining ones for the current behavior.

**Factory Branching**  
This technique uses a feature toggle to control how an object is create using teh Factory Pattern. It is sort of a hybrid of the first two techniques. You control how objects are created from a central location, but you use conditional logic from within the factory class to control the behavior.

The point here is that there are many techniques that can be used to control how a feature is toggled. Choose the one that makes the most sense for your scenario. 

### Retiring Feature Toggles

As Fowler points out, these toggles we introduce into the code can become a problem if not managed properly:

> Release Toggles are transitionary by nature. They should generally not stick around much longer than a week or two...

As part of your process, you should ensure that each release feature toggle is retired, that is, removed from the codebase when it is no longer needed. In the above example, once the new feature is released in production, there is no need for the toggle to exist. Is it making your code more complex than it needs to be. 

### Summary

Release Feature Toggles are a great technique for deploying code but not releasing it for users to access. Without them, trunk-based development wouldn't be possible.

