---
layout: post
title: 'Eliminating Waste with .NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In the last post I wrote some ideas about eliminating waste when developing software. With this post
I'd like to specifically look at waste when developing .NET Core applications.

<!--more-->

## Valuable Code

When developing any application, you want to eliminate waste in order to provide the most value with
the least about of effort. With .NET Core, I have found there are a number of places where waste typically
occurs. The first step is being able to recognize waste. [Mary and Tom Poppendieck wrote about the following
forms of waste][lean-book]:

1. Partially done work
1. Extra features
1. Relearning
1. Task switching
1. Waiting
1. Handoffs
1. Defects
1. Management activities

What I'd like to explore is waste at a more technical level.

### While you were coding

While coding, you introduce waste whenever you don't provide value. I have tried to look at waste
in two forms:

- Activities that I should do more quickly
- Activities that I should automate

While the software development industry is relatively young, there are some well understood
practices to be more productive. The next sections discuss a few techniques I have used
to be a more productive coder.

### Context

While these techniques have worked well for me, they may or may not work in every context. Blindly
following practices is never a good thing. You must make the best decisions you can in your given
context. So keep that in mind. Keep looking for what works best in your world and strive to improve
every day.

## Following an Agile Process

When I think about waste in my process, it typically boils down to how I manage my work. I try
to use an agile approach which to me means:

- Working in small increments
- Release changes frequently
- Get feedback as soon as possible
- Incorporate what I learn into my process

The longer I work on a task, the more nervous I feel about it. I want to feel like I'm on
the right track. If I go in a direction that doesn't provide value, I need to know as soon as
possible to reduce/eliminate wasted time/effort. And that is what agile means to me: keeping
on the right track.

So what does this mean for .NET Core? Keeping on the right track means the cycle I use for start
a new feature/change needs to be as short as possible. I'll create a new class, unit tests covering the
important parts of the code and then share the code with other developers.

I eliminate waste by keeping these shared pieces as small as possible. I used to work differently
and create a lot of code changes that weren't cohesive. I'd combine new features with refactorings
and bug fixes. Now I keep each set of changes to only one of these things. If you spot something
that needs to be refactored, create a task (or as simple as a TODO in the code) to come
back and fix it when you have the current set of changes done. If you find a bug, then log it
as a bug and come back to work it as a separate set of changes.

## Using Coding Standards

Consistency is an important thing in my world view of software development. Consistency eliminates
the need to think about too many things at once. If you perform tasks consistently, you can
learn to do them more quickly, which eliminates waste.

With the currently available IDEs (Visual Studio, JetBrains Rider, etc.) most coding standards can be
enforced automatically. This is a significant time saver as you no longer need to review yours, or a
coworker's code for standards.

That being said some standards still need a human to review. Most notably, naming conventions and code
organization. It is both easy and difficult to eliminate waste when using naming conventions. Easy, because
once you have established a convention, it is simply a matter of discipline to adhere to the conventions. It
is difficult however to _actually establish a convention_.

The best set of guidelines I have found is to make naming things easier are:

- Use a _[Ubiquitous Language][ubiquitous-language]_.
- Use a _[Screaming Architecture][screaming-architecture]_ approach for organization.
- Use functions/properties to convey purpose, rather than comments.
- Don't use 'Core' in any solution/project/namespace/class name, unless it is referencing something .NET Core specific (too confusing otherwise). Use 'Base', or 'Common', or 'Shared' instead.
- Prefer the use of _[Value Objects][value-objects]_ over primitive types.

## Tools

As with most endeavors, there is usually a good tool that will help make the work easier to accomplish. .NET
Core has more than it's share of such tools:

- Microsoft Visual Studio (Windows or Mac)
- Microsoft Visual Studio Code
- JetBrains Rider
- Microsoft .NET CLI
- Git
- xUnit/NUnit/MSTest

It doesn't matter which of these tools you use, however, your familiarity and competency with them does. So
learn the tool you prefer _extremely_ well. Know it inside and out. Learn to get the most out of it.

### Use Your Tools Effectively

Here are some tasks I would expect you to be able to do with your eyes closed:

- Create a new solution/project
- Add a project to a solution
- Add a specific version of a NuGet package to a project
- Create a new file/directory
- Format the contents of a file
- Rename a file/directory (and refactor all the references)
- Run the code locally
- Debug the code locally
- Create a new unit test
- Run all unit tests
- Integrate your code with Git (incl. a proper `.gitignore` file)
- Checkout a branch with Git
- View changes to the current Git branch
- Commit your changes to the current Git branch
- Push/Pull/Merge your changes with a remote Git branch

**NOTE** Ideally, you should be able to do all the above using only the keyboard. Seriously. Really.

### Search and Replace

You will find that the search and replace functionality in most tools is pretty good. However, sometime
you will need the ability to search multiple solutions/projects for something. IDEs aren't a good choice
for that sort of thing. I use a simple tool called [Search and Replace][search-replace] and it has
served me well for years.

### WinMerge

Outside of Git, you will also find times when you need to compare files or directories. I use
[WinMerge][winmerge] and recommend it without reservation.

### TextPad

Even though IDEs will do a great job with text manipulation, sometimes a simpler tool is better. I
use [TextPad][textpad] as my tool of choice. Not only does it support multiple tabs and great snippet
and syntax highlighting, it also has wonderful regular expression search and replace capabilities.

## CI/CD Pipelines

When I first began working with .NET Framework projects, I wanted to automate as much as I could. I had
read a great book series from the Pragmatic Programmers which included [Pragmatic Project Automation][pragmatic-project-automation], a book describing how to build and deploy Java applications using
CruiseControl. I was able to adapt the ideas into a series of build scripts using Nant and it
made it possible to build and deploy software without much difficulty.

In today's world, Continuous Integration (CI) and Continuous Delivery (CD) are the _de facto_ industry
standards for automating the build and deployment steps for software development. I have used a few
tools in the past three years and they all work fine. So choosing the tool is not as important as
actually committing the time to invest in a CI/CD pipeline.

In my experience, it is such a powerful technique that I can't imagine working on a non-trivial
software development effort without a CI/CD pipeline. It eliminates so much wasted effort. You never have to
think about getting your code changes into the hands of your users. It all just happens automatically.

Setting up a CI/CD pipeline is by no means trivial. In fact, it can take a significant effort, depending
on the complexity of the software. For example, setting up a CI/CD pipeline for a single website application
may only take a day or two. And you have to budget for that time. However, if you are setting up a microservices
architecture with several services, a lot more planning is necessary and can take a week or more to get up
and running.

That being said, it is my opinion that this time is well worth it because it will reduce the time necessary
to get code changes in the hands of your users more than any other technique I know.

## Refactoring

Refactoring your code is an important activity to eliminate waste. Basically, you use refactoring techniques
to reduce the amount of code you have:

- Remove code no longer needed.
- Reuse duplicated code.
- Remove comments by using clear naming conventions and straightforward, simple coding practices.

I have used JetBrains Resharper in the past and felt it was more trouble than it was worth. Recently I
have returned to using it, but in a way much more pragmatic to my needs. By biggest complaint was how it slowed
Visual Studio down to a crawl. I discovered that it was mostly due to how it's intellisense works. So I turned
off the Resharper intellisense in favor of the Visual Studio defaults. This made things work much smoother.

In Visual Studio 2019, the refactoring tools are much improved (as with each successive version of Visual Studio) and
it may be possible to forego the use of Resharper and still effectively eliminate waste. But at this present
time, I find that using JetBrains Rider (with integrated Resharper support) provides the most productive environment
for seeing waste and eliminating it.

## Unit Tests

I can't say enough about unit testing. Whether it be test-driven design or simply writing unit tests to demonstrate
behavior of existing code, it doesn't matter. Simply stated, write the tests. Write as many as necessary to confirm
the existing behavior of the code. Write tests to your future self to remind them how things work. It can eliminate
so much waste:

- You can see when changes break existing code (when their tests no longer pass)
- You can find bugs in your code more quickly (when you explicitly test the behavior)
- It affects the design of the code making it easier to maintain

I prefer using xUnit, but I have used NUnit and MSTest as well. I don't believe there is a wrong choice
here. Just write the tests.

## Summary

I have looked at a lot of different aspects of .NET Core and where waste can occur and how to eliminate it. I
hope this serves me well in the future as I refer back to this post to ensure I continue to eliminate
waste from my development of .NET Core apps.

[lean-book]: https://www.amazon.com/Lean-Software-Development-Agile-Toolkit/dp/0321150783
[ubiquitous-language]: https://www.agilealliance.org/glossary/ubiquitous-language
[screaming-architecture]: https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html
[value-objects]: https://enterprisecraftsmanship.com/posts/functional-c-primitive-obsession/
[search-replace]: http://www.funduc.com/search_replace.htm
[winmerge]: https://winmerge.org/
[textpad]: https://www.textpad.com/
[pragmatic-project-automation]: https://pragprog.com/book/auto/pragmatic-project-automation
