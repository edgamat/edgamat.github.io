---
layout: post
title: 'Coding Minimalism'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I believe and promote the philosophy of Coding Minimalism. Let's explore what it means and how it affects developing code.

<!--more-->

## What is Minimalism?

To quote [The Minimalists][minimalists]:

> Minimalism is a tool to rid yourself of life’s excess in favor of focusing on what’s important...

With some creative license, I would offer the following definition of 'Coding Minimalism':

> _Coding Minimalism_ is a tool to rid code of excess in favor of focusing on what’s important.

## How Does Minimalism Apply to Software Development?

Let's start this discussion with stating a truth about software:

**CODE IS A LIABILITY**

More code means more risk.  
More code means more complexity.  
More code means more cost to maintain.

One should always strive to minimize the amount of code  required to achieve the expected behavior. Coding Minimalism is a philosophy that embodies this approach to software development.

## In Practice

As a philosophy, minimizing the amount of code you write is a fine goal. But how does it work in practice? The day to day activity of adding and changing code can be a messy, iterative process. So how does one practice minimalism?

### Recognizing Excess

The first step in minimizing the code you write is recognizing when code is not necessary. Here are some examples:

Un-necessary parentheses:

```csharp
var isActive = (user.Status == Status.Active);

// Less code:
var isActive = user.Status == Status.Active;
```

Un-necessary processing:

```csharp
var isActive = user.Status == Status.Active ? true : false;

// Less code:
var isActive = user.Status == Status.Active;
```

Un-necessary conversion:

```csharp
var count = 1;
order.LineCount = Convert.ToInt32(count);

// Less code:
order.LineCount = count;
```

And as some of my co-workers know, here's my favorite regarding strings:

Extra Stringy:

```csharp
var name = "sample";
order.Name = Convert.ToString((string)name.ToString());

// Less code:
order.Name = name;
```

### Refactor Your Code

Refactoring is the _art_ of restructuring existing code, without changing its external behavior. To me, it used to seem like a luxury, but I see it now as a necessity to embracing minimalism when coding. 

Refactoring code allows you to see patterns and remove excess where possible. As with most things, it takes practice to do well. And it isn't something we typically dedicate a lot of time to, so it is best to be good at it. So practice, practice, practice :-)

### YAGNI Principle Applies Here

The [YAGNI concept][yagni] is something that has been around for over 20 years. It stands for:

YAGNI = You Aren't Gonna Need It

Keeping a minimal codebase means don't add more code than is necessary to accomplish a task. YAGNI is a great guide for keeping things simple (whish should mean less code).

### KISS Your Code

I try to follow an Agile methodology that requires small, focused sets of changes to the codebase. While I used to be the worst offender of this, I no longer mix several things together in a single set of changes. It is just too confusing and complex. 

The principle of KISS (Keep It Simple, Stupid) could be the best part of a minimalist attitude when coding. It states that most code will work best when it is kept simple, and not complicated.

I have produced changes to code, confident that my solution was the best possible. But my co-workers would often point out simpler solutions that would be just as effective. It is a humbling experience, but it is very kind of my co-workers to offer such advice. Keep it as simple as possible.

## Don't Go Overboard

The practice of Coding Minimalism shouldn't result in un-readable or un-maintainable code. If that is what you end up with, then you're doing it wrong.

You should still maintain proper coding conventions, formatting rules, unit tests and so on. Coding Minimalism is not a sledge hammer used to pound everything, but rather a way of looking at your code and assessing it as possibly having excessive parts that could be removed.

A typical example I see is code where statements are excessively long. A single statement tries to do too much. It would make the code much easier to understand if the statement was split into two or more statements. 

This may seem counter intuitive to the concept of Coding Minimalism. But remember, that Coding Minimalism does not mean code with the fewest number of statements as possible. 

The mindset should always be to strike a balance between the fewest statements, functions, classes as possible, while still adhering to good programming practices and conventions.

### Communication is Key

The more challenging part of Coding Minimalism is knowing when, and when not, to remove code. One developer might see a statement as un-necessary, while another may not. One might see a statement as readable, while another might struggle with its complexity.

My approach to this challenge is to be mindful of the context. If the code is not clear to everyone involved, then it is a great opportunity to learn and grow. So talk about it. Offer alternatives. Ask for clarification. But don't over-do it... you most likely get it perfect every time.

You may find that over time your ability to see excessive coding practices improves, as does your ability to find simpler solutions to problems. So don't sweat it if things aren't perfect. The journey is just as important as the destination.

### Tools Can Help, Sometimes

I develop a lot of code using JetBrains Rider. It, like many modern IDE tools, assists developers with highlighting issues. Spelling mistakes used to be my worst enemy (still are to be honest). I rely on Rider to highlight these typos and it allows me to spot them and correct them right away. Another feature it provides is highlighting dead code.

Spotting the code that is not used (ie. dead code) is one of the better features in an IDE. I rely on it a lot to help me spot things that can be removed. Or, as it can sometimes be, to recognize a mistake that has been made.

The tools we use can help us a lot in producing code with as few lines as possible. JetBrains Rider includes the whole suite of Resharper refactorings as well. However, this is where you can get into trouble. Some of the refactorings may result in fewer lines of code being written, but it doesn't always mean the code will remain easy to understand and maintain. It takes practice (are you seeing a pattern here!) to use these tools effectively. 

## Summary

Coding Minimalism is a practice of removing excess waste from a codebase. If you practice it more and more, you will see these excesses more easily. But you must always strike a balance between fewest lines of code and proper coding practices like formatting and readability. 

[minimalists]: https://www.theminimalists.com/minimalism/
[yagni]: https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it
