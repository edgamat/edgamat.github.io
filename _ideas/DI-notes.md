
In most applications you will encounter cases where one component depends on another component and it depends on yet another component. This is called an "object graph" and represents all the components and their relationships.


Somewhere in the application code, there needs to be a place where the objects are created. A request is made to the application to create an instance of the `CustomerController`. The application needs to know how to create an instance of the `ILogger<CustomerController>` interface as well. In a large application there can be dozens or hundreds of such dependencies. 

### Composite Root

The best practice is to define a logical location in the code where the objects are composed together. This is called the "Composite Root", and should be placed close to the entry point to the application (because the objects the application needs are composed there).

In the .NET 5 ASP.NET project templates, the `Startup` class has a method called `ConfigureServices`. This is the Composite Root for the application. All the rules defining what implementation to use for each dependency are defined here using a DI Container.

```csharp
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
    }
}
```

### DI Container

A DI Container is a library that provides dependency injection functionality that automates the task of composing objects, as well as intercepting and managing the lifetime of the objects it creates.

Microsoft provides a default DI Container, and it allows for third-party DI Containers to be used as well. The `IServiceCollection` represents the list of all the objects the DI Container may be asked to create. 

When an ASP.NET application receives a request, it uses the configured routing settings to choose the controller used to process the request. It uses the DI Container to create an instance of the controller and any of its dependencies. 

In our example, the DI container knows that in order to create a `CustomerController`, it needs to have an object that implements the `ILogger<CustomerController>` interface to pass into the constructor of the `CustomerController`. The DI Container finds an instance of this interface to use, and if it can't find one, the DI Container creates an instance.

The DI Container keeps track of all the objects is has created and disposes of them when they are no longer being used. 



### Object Lifestyle

An object lifestyle defines the intended lifetime of a dependency (when they are created and when they are disposed of). Three lifestyles are typically implemented in most DI Containers:

| Name | Description |
| - | - |
| Transient | Objects are created each time they're requested |
| Singleton | Objects are created the first time they're requested and a single instance exists while the application is running |
| Scoped | Objects are created once per 'scope'. They behave like a singleton within the defined scope |

In an ASP.NET application, a separate scope is created for each web request being processed.

When you register a class with the DI Container, you must define the lifestyle is should use. The built-in DI Container includes methods to define these:

```csharp
services.AddTransient<IMyInterface, MyImplementation>();
services.AddScoped<IMyScopedInterface, MyScopedImplementation>();
services.AddSingleton<IMySingletonInterface, MySingletonImplementation>();
```
**NOTE** A class registered with a Singleton lifestyle does not need to implement the Singleton Design Pattern. It just has to be thread-safe.

Transient and scoped objects are disposed as soon as the scope they are created in is disposed. Singleton objects are disposed of when the application shuts down.

**How do I choose?**

As you can imagine, choosing the proper lifestyle for a dependency is very important. If you choose the wrong lifestyle, it can cause bugs that are typically hard to diagnose.

Singleton dependencies typically have to be thread-safe because there may be many simultaneous calls being made to them from different threads in your application. Taking our earlier example, the implementation of the `ILogger<CustomerController>` is thread-safe. Each web request being processed by the application has access to the same logging implementation. 

Scoped dependencies are a blessing and a curse. They can make some problems easier to solve, but can make the code more difficult to understand and reason about.




[Dependency Injection Principles, Practices, and Patterns][dibook2] by Mark Seemann.



[dibook2]: https://www.manning.com/books/dependency-injection-principles-practices-patterns




Take for example the Entity Framework DbContext class. Applications that implement one of these classes to access a database typically register them using as Scoped lifestyle. That means that a separate DbContext is created for each web request processed by the ASP.NET pipeline. This is good, because the DbContext class is not thread-safe. 

If your controller depends on classes that access the database, each of these classes will have the same instance of the DbContext class injected into their constructors. This can lead to confusion when the classes are trying to access the database. 


Interception

Interception is the ability to intercept calls between two collaborating
components in such a way that you can enrich or change the behavior of
the Dependency without the need to change the two collaborators themselves.

