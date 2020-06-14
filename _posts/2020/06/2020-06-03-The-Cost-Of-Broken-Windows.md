---
layout: post
title: 'The Cost of Broken Windows'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In the book [The Pragmatic Programmer][pragprog] I was first exposed to the idea of "broken windows theory" and how it impacts the life of a software product. I was reminded of that this week and it is still an important concept to consider in software development.

<!--more-->

## The Theory

The [broken windows theory][theory] that states that visible signs of neglect, e.g. broken windows on a house, encourages further neglect. In _The Pragmatic Programmer_, the authors suggest that programmers need to fix bugs, bad choice, bad designs, etc. and not leave them unrepaired.

When I look at a codebase for the first time, I have my own set of (admittedly biased) options on the quality of the code. For example, I clone a repo of C# code and compile the code. If I can do that without any compile issues, all dependencies are found and, well, it just works, then I am left thinking the code is in good shape. This occurs less often than I'd like. Most of the time, you'll encounter some problem that prevents you from getting a code base installed on a new workstation.

## Types of Broken Windows

Neglect can appear just about anywhere in a codebase. The "broken windows" of the software world are too numerous to give a complete list. However, in my experience, there are some that are more costly than others. Left unrepaired, they can cost a significant amount of time and effort. So here are some that I feel strongly about fixing as soon as possible once I discover them.

### Failing To Compile

Let's start with an easy one. If someone commits a change to the codebase which causes it to not compile on other's workstations, it needs to be fixed as soon as possible. In some teams, the standard is to stop all other work until the codebase is back compiling correctly. I stress this as much as possible. When I encounter a branch of the code that won't compile, I'll gather the entire team to discuss and correct the issue and stop all other work until it is resolved. I feel it is important for the everyone to participate. They can see first hand what caused the issue and how to avoid it. In addition, it is a great opportunity to work 'as a team'; practice communicating in real-time, divide work efficiently, have a sense of pressure to resolve an issue. All things that are good to practice.

Having an automated continuous integration (CI) pipeline can expose these issue almost immediately. When it wasn't possible to achieve a CI pipeline, I would resort to compiling the code myself each time someone introduced a change. That simple thing can expose a lot of issues quickly. No one should ever have an excuse for letting code sit in an un-compilable state.

### Failing Unit Tests

Assuming the code includes an automated suite of unit tests, ensuring they all are passing is paramount. A failing test must be corrected as soon as possible. Full stop. No joke. Some teams are fortunate to have automated integration tests. Keeping them in a 'green' state with no failures is incredibly important. It's why so much effort is put into them in the first place. If you don't focus on fixing failing tests then why bother writing them at all?

### Compiler Warnings

This one can be difficult to justify, but I feel that a constant set of compiler warnings each time you compile your code is a problem that must be addressed. If you have no warnings and one pops up as you work a problem, you can spot it easier than if you have a pre-existing set of 25 and you add one more to make it 26. 

I have compiled C# projects for the first time when joining a team and encountered hundreds of compiler warnings. I asked the existing code maintainers how to correct them (thinking I had something wrongly installed) only to have them say "Oh, that's normal.". No, that is not normal. It is a sign of neglect. It didn't start this way. Over time, one at a time, these warnings appeared and were never addressed. And now they are so out-of-control, that no one knows what they all mean as far as the code quality is concerned. Don't let this happen. When a warning appears, address it. Can it be ignored? Then add an override to remove it from the compiler warnings moving forward. If not, fix it.

### Coding Standards

Each team should follow an agreed-upon set of coding standards. Most of the time, what these standards are isn't as important as adhering to them. If everyone says they will follow the standard, then when someone strays from it, it is acceptable to call attention to it and get it resolved. However, if you don't have standards or if no one enforces the standards, then the code will start to look neglected. And that neglect will spread and the quality of the code will suffer for it. 

For example, on a C# project, the team might decide to ensure all `using` directives are ordered alphabetically. Another team may choose to order all the `System` namespace `using` directives first, and all others after, each group being sorted alphabetically (This is how Visual Studio would order things so people have used this approach for years... old habits...) Which approach is taken isn't critical. But if someone doesn't follow the standard, it should be corrected and not left in the codebase.

In today's world, almost all programming tools and frameworks support the enforcement of standards. It might be something to consider when you evaluate a potential new tool. Can I ensure it follows a standard? If the answer is no, then it may not be mature enough to use on 'real' projects.

## Summary

Signs of neglect in your codebase can be seen almost anywhere you look. Leaving these issues unrepaired will lead to further neglect and that will have a negative impact on productivity as a project progresses. Make it a priority to fix your "broken windows" in your code, whatever they may be. You'll be better off for doing it.

[pragprog]: https://pragprog.com/book/tpp/the-pragmatic-programmer
[theory]: https://en.wikipedia.org/wiki/Broken_windows_theory
