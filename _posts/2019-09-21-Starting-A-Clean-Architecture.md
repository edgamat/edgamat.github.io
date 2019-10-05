---
layout: post
title: 'Starting a Clean Architecture'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Robert C. Martin (Uncle Bob) has written at length about what he calls a [Clean Architecture][clean-arch-blog]. For
awhile I wanted to explore these concepts in C#. Specifically a REST API built using ASP.NET Core. This is the first
in a series of posts describing my experience.

<!--more-->

I want this new approach to building software to achieve the following goals before I am done:

- First, I want do demonstrate The Dependency Rule, that in a layered architecture, references can only point inwards.
- Second, I want to use the Clean Architecture principles to build an application.

## The Principles

These principles are laid out in the book [Clean Architecture][clean-arch-book]:

- Be framework independent: Don't allow the core model of your architecture to depend on a framework.
- Be testable: Ensure the essential business logic of your application can be tested without any external dependencies.
- Be UI/Database independent: The business logic should not be bound to UI or database decisions.
- Scream what you are: The high-level structure of your application should scream what it is, not what the framework is.

Using ASP.NET Core to accomplish this goal is important to me since it is the tools I use for the majority of the
applications I am asked to write at my job. Using these tools is always evolving. I have been using ASP.NET Core since its
initial release. However, the way I write applications today is dramatically different than when I first started. I expect
this exploration into using a Clean Architecture will change how I write applications as well. Or at least I hope so.

## Independence

Reading the list of goals and principles you can see that it revolves around the notion of independence. While it is not
possible (or desirable) to build an entire application to be 'independent', it is very important to maintain as much
independence as possible with the core business logic of your application.

## The Book Warehouse Application

To demonstrate this architecture, the application will be a Book Warehouse:

- The REST API will provide access to the **Warehouse** of books.
- Users will be able to query the Warehouse to determine if a **Book** is in stock.
- Users will be able to create an **Order**.

That's enough to get us started.

## Where to start

I have been told from an early age that one 'builds' software, like a house. It is constructed. I feel differently. It is
too organic a thing to be confined to such ridged notions. Software _grows_.

You start small and learn from what you experience to achieve a working software application. So where does an application start to grow? It starts with the Domain business logic. This set of entities represents the business objects in
your particular subject domain, including general and enterprise-wide rules that the domain operates under.

I advocate starting here in the center, with these entities. I do not advocate starting with the database or
with the user interface. Nor do I advocate starting with the framework (in this case ASP.NET Core) the application
uses. Start with the domain entities, get them right. Test your assumptions of how the rules work.

**NOTE**: That's not to say that data modelling and user interface design and exploration aren't important. They are
and typically are done in parallel with the design of the business logic. But when you are design your UI and database,
there can be a lot of volatility at first. Stick with the Entities to ensure you know what the application is all about.

## Setup the Domain Logic

With ASP.NET Core, I will start with a .NET Standard Class Library. It begins its life with no dependencies except the .NET base classes and the C# programming language. I will create a minimal folder structure for what I know I will build:

- a source code folder: `/src`
- a test case folder: `/test`

Due to the size of this effort, the entire codebase will be contained in a single .NET Solution. Here are the commands
used to create this initial structure:

```powershell
$projectName = "BookWarehouse"

md "$projectName"

# Ensure the SDK is locked in so the solution is independent of other SDK versions
cd "$projectName"
dotnet new globaljson --sdk-version 3.0.100
cd ..\

md "$projectName\src"
md "$projectName\src\$projectName.Domain"

# Create the class library for the Domain business logic
cd "$projectName\src\$projectName.Domain"
dotnet new classlib --no-restore
cd ..\..\..\

md "$projectName\test"
md "$projectName\test\UnitTests.$projectName.Domain"

# Create the class library for the Domain business logic unit tests
cd "$projectName\test\UnitTests.$projectName.Domain"
dotnet new xunit --no-restore
dotnet add reference "..\..\src\$projectName.Domain\$projectName.Domain.csproj"
cd ..\..\

# Create the Solution file
dotnet new sln -n "$projectName"
dotnet sln add ".\src\$projectName.Domain\$projectName.Domain.csproj"
dotnet sln add ".\test\UnitTests.$projectName.Domain\UnitTests.$projectName.Domain.csproj"

# Create a .gitignore file
$url = "https://raw.githubusercontent.com/github/gitignore/master/VisualStudio.gitignore"
$output = ".gitignore"
Invoke-WebRequest -Uri $url -OutFile $output

# Initialize the Git repository for the code.
git init

cd ..\
```

## Next Time

This is a good start. We have a minimal .NET Solution for our Domain business logic. Next we'll start
adding in some functionality for our application and see where it leads us.

[clean-arch-blog]: http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
[clean-arch-book]: https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164
