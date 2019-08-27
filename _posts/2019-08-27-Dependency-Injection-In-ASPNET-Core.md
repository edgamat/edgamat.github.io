---
layout: post
title: 'Dependency Injection in ASP.NET Core'
published: false
---

# Dependency Injection in ASP\.NET Core

Dependency Injection (DI) was something I read about several years ago and I didn't really understand it well until I encountered ASP\.NET MVC. Specifically, the way [Autofac][autofac] made it all work. I had used other DI containers, but it was using Autofac that really made it clear to me.

I also found a great resource to help me understand it further. The book was [Dependency Injection in .NET][dibook] by Mark Seemann. Recently a second edition was published titled [Dependency Injection Principles, Practices, and Patterns][dibook2]. I was curious to see how 8 years of experience between these publications had influenced the author's view of DI.

The second edition is more than simply a DI book. It contains great advice on good object-oriented design, and how to write loosely-coupled code.

So how does this fit into ASP.NET Core?

Over the past year I have worked on a few ASP.NET Core REST API services and they all used DI to compose the application. I learned a lot and wanted to write some of these ideas down for future reference. Here goes nothing.

## Should I use Autofac?

The built-in DI container in ASP.NET Core is simple to use and configure. It implements three lifetime styles that allow you to implement a solid DI framework:

| Name      | Description                                                                                                        |
| --------- | ------------------------------------------------------------------------------------------------------------------ |
| Transient | Objects are created each time they're requested                                                                    |
| Singleton | Objects are created the first time they're requested and a single instance exists until the application shuts down |
| Scoped    | Objects are created once per request                                                                               |

While this makes sense for ASP.NET web requests, it is more challenging when you more complex scenarios such as background services and message queues.

[autofac]: https://autofac.org/
[dibook]: https://www.manning.com/books/dependency-injection-in-dot-net
[dibook2]: https://www.manning.com/books/dependency-injection-principles-practices-patterns
