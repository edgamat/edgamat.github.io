---
layout: post
title: 'Functional C#, Revisited'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

A few years ago I dipped my toes into the world of functional programming in C#. Let's revisit that idea.

<!--more-->

### A Bit of History

The primary development language I use is C#. I am by no means an expert, but I would say that I have 
an above-average working knowledge of using C# to develop business software. It was conceived as a general purpose
language focused on the object-oriented programming paradigm. Over the years, functional programming principles
have crept into the language and by many accounts, you can consider C# pretty capable from a functional programming
perspective. ([Watch this video for some context](https://www.youtube.com/watch?v=CLKZ7ZgVido))

When C# 7 was released, I watched a few Pluralsight courses on functional principles and how to implement them in 
C#. I tried the ideas out, but only a few made their way into any production code I was writing. I 
tried using a "Result" class to avoid using exceptions once or twice. I was more aware of striving to make
immutable functions. And I was more aware than ever of the evils of nulls. In the end however, there was never a 
formal attempt to say "I am using functional principles when coding in C#."

A lot has occurred since C# 7 was released. Several new features have been added to the language and I am curious 
to see how that would influence the functional programming aspect of C#.

### Functional Principles

Let's first describe what I mean when I use the term functional programming. Or maybe more to the point, what do I
mean when I want to apply functional principles to C#? In my mind, here are the principles from the functional
programming world that I want to embrace with my C# code:

- **Immutability** I want to prefer immutable structures in the code that avoid changing state. The expected
benefits are more readable code and code that is easier to understand how changes are made.

- **Honesty** I want to avoid throwing exceptions in the code. I want to create honest (precisely defined inputs
and outputs) functions that remove the need to look at the implementation to know how a function might behave.

- **Type Narrowing** I want to avoid primitive obsession by narrowing the set of values a variable can hold, reducing 
the need for type checking that can pollute a codebase.

- **Missing Values** I want to write code that deals with missing values (like nulls) is a first-class manner,
removing an entire category of bugs from the code.

There are many other aspects of functional programming I am leaving out here. Using functions as first-class citizens,
for example. Higher-order functions, the concepts of Map and Bind, and curried functions are not something I'm focusing
on at this time. But I hope to get there eventually.

### Nullable Reference Types

Dealing with nulls is always challenging. IDEs like Microsoft Visual Studio and JetBrains Rider do a great job
of highlighted cases where you may encounter a null value and need to check explicitly for null. But in C# 8,
nullable reference types were introduced. I want to explore this in detail. 
- Does this 'fix' nulls or is it something added to the language too late? 
- Does it replace the need for the functional concept of "Option" or "Maybe", or are these still concepts 
  that make sense to include in your toolbox?

### Records

Immutability in C# is rather non-existent. C# 9 introduced the idea of records, which allow you to make custom
immutable types much easier. C# 10 provided a few enhancements allowing record structs (rather than record classes).
- Are these really the immutable foundations we have been looking for? 
- Does using records confuse the reader when comparing them to classes? How immutable are they? 
- Do they make dealing with primitive obsession easier, with less boilerplate code?

### Pattern Matching

C# 8 began a journey of new pattern matching features that have been tweaked with the subsequent versions of C#. 
They provide a rich and more concise means of expressions that make switch statements look childish. 
- How useful are they in practice? 
- Are they foreign enough to make C# code too confusing? 
- Are they easy to learn and apply? 

### Applying Functional Principles

I look at these new languages features from the perspective of functional programming. I wonder if there is enough 
'critical mass' for a set of functional concepts to be used on all C# projects, providing more clear code, with 
fewer bugs. I plan on exploring these in more detail over the next few months. 

I write a lot of Web API projects in C# for business applications. My goal is to introduce functional principles
and evaluate how effective they are. Specifically, I am going to investigate the feasibility of:

- Using the `Option<T>` monad as a means of dealing with missing data. I'd like to explore how far you can go using
the nullable reference types without introducing an explicit `Option` type. 

- Using `Result`, a special case of the `Either` construct. How far can we move away from exceptions in our code?
Is this something that will make the code significantly better? 

- Using `Record` types more and classes less. Does this improve things or make things worse? How does immutability 
impact design? Does the code end up being a confusing mix of records and classes? 

- Using the new pattern matching capabilities to simplify complex logic. Some of the examples of the new patten
matching syntax look foreign and confusing at times. Is that a long-term truth or simply a manifestation of being
unfamiliar with the power they offer?

There's a lot to uncover here. I look forward to what I might learn and be able to apply to the work I do. 
