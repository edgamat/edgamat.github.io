---
layout: post
title: 'Clean Architecture Ideas for Microservices using ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I have written about a [Clean Architecture]({% post_url 2019/09/2019-09-21-Starting-A-Clean-Architecture %}) on several occasions. While I don't see eye-to-eye with its popular proponent Robert C Martin on several ideas he has, the use of a Clean Architecture is not one of them. It is a effective approach to creating any software application. Over the last year I have worked with a set of microservices written in ASP.NET and I thought I would write down some of the ideas I had regarding the use of a Clean Architecture in this context.

<!--more-->

### What's So Special?

One goal of a Clean Architecture is to keep the business logic framework independent. In other words, the core business logic of your application should not be dependent on your choice of database, or user interface choices. The microservices I have worked with have also involved a few other key framework dependencies that also should be removed from the core business logic:

- HTTP requests, via the `HttpClient` class
- Messaging, via a message broker, such as [MassTransit][MassTransit]
- File Creation, via the Windows file system
- Data Serialization, via JSON, such as [Newtonsoft.Json][Newtonsoft]
- Date/Time instances, via the `DateTime` class in the .NET runtime
- Report Generation, via [Crystal Reports][CrystalReports]
- Email Messages, via a hosted SMTP Service

A couple of these, namely Report Generation and Email Messages, are hosted via separate microservices. They are accessed via HTTP requests or publishing messages, via a message broker. But the remaining dependencies need to be addressed properly to maintain the proper framework independence in the business logic.

### Assumptions of Clean Architecture

Using ASP.NET Core, the assumption I make for a Clean Architecture is to layer the applications similar to this:

Layer/Namespace | Purpose
---------|---------
MyApp.Domain | To store the core business logic of the application
MyApp.Persistence | To implement logic to persist data to and retrieve data from a data store.
MyApp.Api | To implement an HTTP API for callers to make requests to the microservice.

**NOTE** There is no universally-accepted guidance on how to split these into separate projects/assemblies. Some teams feel each deserves its own project, while others don't. Whatever your preference, the important thing to remember is to keep proper independence of each concern. Any mixing of these concepts in the code makes achieving the benefits of a Clean Architecture difficult.

### HTTP Requests

The standard mechanism in ASP.NET Core for making HTTP requests is the `HttpClient` class, created via the `IHttpClientFactory`. The factory is configured in the `Startup` class. This holds true for all .NET project types, not only ASP.NET Core. The 'Clean Architecture' way of utilizing this is no different than how persistence should be tackled. An abstraction layer is defined in the Domain layer and its implementation is defined outside of this layer. If you are using the HTTP services as persistence, then it makes sense to include this within the application's Persistence layer. But you amy also choose to place the implementation in the API layer as well. The API layer includes all the HTTP functionality one could want, so placing the HTTP implementation details in this layer makes sense as well. 

The abstraction in the Domain layer is an interface definition for the functionality:

```csharp
public interface IOrderStorageService
{
    Task<Result> CreateOrderAsync(Order order, CancellationToken token);
    
    Task<Result> CancelOrderAsync(Order order, string reason, CancellationToken token);
    
    Task<Order> GetOrderAsync(Guid orderId, CancellationToken token);
}
```

In fact, in some scenarios, the abstraction looks identical to the 'standard' Repository pattern. 

The implementation is kept out of the Domain layer and that is the important bit here. 

**Side Note**  
> The projects that I have been working on don't do this particularly well. More often than not you'll find HTTP requests and interactions with the persistence logic intermingled with the core business logic. Not our finest hour. But it has taught me a valuable lesson about the side effects of these decisions. The code is much more complicated and more challenging to test. Introducing these concepts now would be a large departure from the rest of the codebase. But there may be some good that may come from attempting such abstractions regardless.

### Messaging

The microservices I have been working with use MassTransit to broker messages between services. MassTransit is also used to place work in background threads for processing. Publishing events or sending commands is done fairly well, from a Clean Architecture perspective. There is an abstract interface in the domain used to issue such messages. However, the Domain layer has a direct dependency on MassTransit (its interfaces are the ones used within the Domain layer). A better approach would be to have a Domain layer interface for this abstraction and use the MassTransit functionality only in the Persistence (or API) layer. 

The other aspect of messaging is consuming the messages. The MassTransit consumer classes are typically live in the Domain layer as first-class citizens. And in hindsight, that was a mistake. These consumer classes would be better lived in the API layer. They should function similarly to Controller classes. They receive the messages, validate them and then pass them along to the Domain layer for processing. The message class is a Data Transfer Object and not a Domain object. It makes sense to map the incoming message to a domain object within the consumer class, just as one would do in a Controller. 

### Other Concerns

Dealing with file I/O and `DateTime` methods are both operating system specific. Your application's business logic should not be dependent on these aspects. While not 'frameworks' per se, they are both areas that make testing your application very difficult. It can be tempting to throw a wrapper around the methods you need:

```csharp
public interface IFileSystem
{
    Task<byte[]> ReadAllBytesAsync(string path, CancellationToken token);

    Task WriteAllBytesAsync(string path, byte[] bytes, CancellationToken token);
}
```

The side effect of this approach is the tendency for the mechanics of the file system will carry over into your domain layer. It is better (in most cases) to abstract the file concept in your domain layer:

```csharp
public struct FileContent
{
    public readonly string FileName;
    
    public readonly byte[] Bytes;
    
    public FileContent(string fileName, byte[] bytes)
    {
        FileName = fileName;
        Bytes = bytes;
    }
}

public interface IFileSystem
{
    Task<FileContent> ReadFileAsync(string path, CancellationToken token);

    Task WriteFileAsync(FileContent fileContent, CancellationToken token);
}
```

There isn't a huge difference in these approaches but the former uses domain concepts rather than how the file system thinks about things.

The same can be said about the `DateTime` static methods. This is one way:

```csharp
public interface IDateTimeProvider
{
    DateTime Now { get; set; }
    
    DateTime UtcNow { get; set; }
    
    DateTime Today { get; set; }
}
```

But this approach does not embody how your application 'thinks' about time. Here is an alternative approach:

```csharp
public interface IDateTimeProvider
{
    DateTime GetInstance();
}
```

Again, it is not a huge difference, but it helps divorce your Domain logic from the frameworks set up by the operating system.

### Summary

Using a Clean Architecture approach with microservices is possible. It presents some challenges, but nothing that can't be dealt with. Mostly you will find you have more framework dependencies for SMTP, HTTP, and Messaging interfaces. Reducing or at least minimizing your dependencies on these frameworks in your Domain layer is desirable and achievable.


[MassTransit]: https://masstransit-project.com
[CrystalReports]: https://www.crystalreports.com
[Newtonsoft]: https://www.newtonsoft.com/json
