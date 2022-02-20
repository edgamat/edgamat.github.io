---
layout: post
title: 'Dependency Injection - Introduction'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Dependency Injection (DI) is an important aspect of many software development frameworks. Let's introduce the topic.

<!--more-->

### Loose Coupling

Coupling describes how tightly a component is related to other components in a software application. It is desirable to create components that make them easy to use by other components and depend as little as possible on other components. This is known as "loose coupling" or "loosely coupled" components.

When components are loosely coupled, it makes them much easier to test. It also makes them much easier to maintain and evolve because changing one component has little or no impact on components it relates to, or depends on.

### What is Dependency Injection

Dependency Injection (DI) is a set of software design principles and patterns that enables you to develop loosely coupled code. 

Within the context of object-oriented design, a dependency is an object that a class depends on to function properly.

Let's look at a scenario where two classes are _tightly coupled_:

```csharp
public class CustomerController
{
    private readonly ILogger<CustomerController> _logger;

    public CustomerController()
    {
        _logger = new FileLogger<CustomerController>($"D:\ApplicationLogs");
    }
    
    ...
}
```

In this configuration, the `CustomerController` depends on an explicit implementation of the `ILogger<CustomerController>` called the `FileLogger<CustomerController>`. It records all the log entries in a file located in the `D:\ApplicationLogs` folder on my hard drive.

If I want to change where the logs are stored on my hard drive, I would need to modify the code of the `CustomerController` class. Or what if I wanted to use a database to store the log entries? I would need to change the implementation of the logger that the `CustomerController` uses. Again, that would require me to change the code of the `CustomerController` class.

What if I wanted different instances of the `CustomerController` to all depend on the same instance of the `FileLogger<CustomerController>`? I can't do that if I am explicitly defining the instance being used within the `CustomerController`.

These two components (the `CustomerController` and the implementation of the `ILogger<CustomerController>` interface) are _tightly coupled_ because changes to one of the components requires changed to the other. This makes the code much more difficult to test. It also makes it more difficult to maintain since a single change may now require changes to multiple components.

Now let's look at a scenario where the two classes as _loosely coupled_. In the following code sample, the `CustomerController` still depends on an implementation of the `ILogger<CustomerController>` interface. But the `CustomerController` class has an implementation of the `ILogger<CustomerController>` passed in (or _injected_) using a constructor parameter.

```csharp
public class CustomerController
{
    private readonly ILogger<CustomerController> _logger;

    public CustomerController(ILogger<CustomerController> logger)
    {
        _logger = logger;
    }
    
    ...
}
```

The implementation of the interface can change and it won't impact the code within the `CustomerController`. This makes these two components _loosely coupled_. 

It also means that when writing a unit test for the `CustomerController` class, I can choose an implementation of the `ILogger<CustomerController>` that best suits the context of the test. I may create a dummy class that implements the `ILogger<CustomerController>` interface but provides no real functionality. Or I may create an implementation that records each time its methods are called so I can verify that the controller interacted with the logger as expected. The point is that I can swap out implementations without changing the code of the `CustomerController` class.

Dependency Injection advocates that when one component depends on another component, that the dependency be _injected_ into the component, rather than explicitly defined. There are different ways to inject these dependencies. In the example I used, the design injects the dependency through the class constructor. But it is also possible to inject a dependency through a method parameter, or a class property. 

### Benefits

In addition to making a component easier to test, injecting the dependency into the class provides other benefits:

- I can control the lifetime of each instance separately. For example, I may want to share a dependency between two or more instances of a component. Injecting the dependency into a component allows this to be possible.

- I can intercept the dependencies at runtime and modify their behavior prior to injecting them into a component. For example, I could take the implementation of the `Ilogger<CustomerController>` and wrap it in a class that writes each log entry to the console, in addition to writing the log entries to a file.

Injecting the dependencies in this manner reduces (or loosens) the coupling between the components. And this will lead to code that is easier to maintain and evolved over time.

### Drawbacks

The largest drawback to injecting dependencies into components is the complexity needed to construct components at runtime. I need to write code to help me create each of the dependencies for a component whenever I need to create an instance of the component. 

Thankfully there are many libraries out there to help implement Dependency Injection patterns in your code. We'll take a look at one in the next article.

