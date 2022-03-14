---
layout: post
title: 'Test-Driven Development Revisited'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I've thought about and written about testing a lot on this blog. Specifically I've written a lot about Test-Driven Development. My thoughts on it have changed recently and I wanted to write them down.

<!--more-->

### In The Past

I have always thought of test-driven development (TDD) as a powerful technique for developing software. I have seen it's benefits with my own eyes and it has improved the design of my code whenever I have used it. 

But I'll admit, I don't use TDD enough in my day-to-day coding. I primarily follow TDD when fixing things. When a bug is reported, I'll use a TDD approach to change the code. But I rarely use TDD when I start a new feature. But that's not true either. I do use TDD the less sure I am of the changes I am making. The larger and more complex the codebase, the more likely I am to use a TDD approach.

In several of his books, Robert C. Martin as promoted TDD as a requirement for professional software development. And I have always thought that this is too heavy-handed. No technique, no matter how good it is, can be universally used in all circumstances and contexts. So I've had my own bias with his writings and his claims on TDD.

In 2021 I read _Code Craftsmanship_ by Robert C. Martin. In the first part of the book, Martin spends a significant amount of time discussing Test-Driven Development (TDD). For Martin, TDD is the most important discipline for professional software development:

> "TDD is the lynchpin discipline. Without it, the other disciplines are either impossible or impotent."
- Page 15, _Code Craftsmanship_

Martin walks the reader through the rhythm of TDD in practice. He even includes videos of what it is like to develop code in a test-driven manner. He describes the so-called "laws" of Test-Driven Development, and a series of rules to aid you in your adoption and practice of TDD.

He makes a compelling argument for using TDD. Out o fall of his books I have read, it is the most thought out and thorough descriptions I have seen him make. 

I took notes; I wrote out all the laws of TDD along with the 14 rules he provides. But yet, I didn't change my approach to software development. I don't know why. Perhaps I lack the discipline to follow a purer form of TDD. In my experience TDD can be difficult. It is something that requires a lot of practice to do well. 

I suppose for me, there was never that 'A-ha!' moment where everything fell into place. I never reached that place where Martin was. I was never able to convince myself to follow a TDD approach to software development. I practiced it, some of the time, but I wouldn't feel the need to use it from the onset of developing new code.

### What Has Changed

I have recently read a new book from Dave Farley called _Modern Software Engineering_. Farley describes a new way of thinking about software engineering. He provides a practical approach to software development, based on his experiences, "that applies a consciously rational, scientific style of thinking to solving problems" (excerpt from the preface). One of the key parts of his approach is test-driven design.

Farley's arguments for using TDD are centered around two key ideas. First, the idea that using TDD is fundamental to exploring, discovery and learning about how to build software. It is at the heart of working iteratively, getting feedback quickly on the direction you are going, and being experimental. Second, the idea that TDD is one of best tools for managing complexity. TDD can improve the design of software, improve the modularity of the code, separate concerns between the modules, and encourages us to create testable code.

Throughout this book, as it explores these fundamental ideas of what software engineering means, Farley describes TDD as an important part of the entire process. I dare say, for Farley, TDD is also a "lynchpin" discipline.

But Farley's thoughts and arguments for using TDD seemed to resonate with me more than ever before. I cannot say if Martin's approaches were any less descriptive and compelling. I think it comes down to me finally having that 'A-ha!' moment. 

### The Future

I would encourage you to read both of these books:

**Clean Craftsmanship: Disciplines, Standards, and Ethics**..
https://www.informit.com/store/clean-craftsmanship-disciplines-standards-and-ethics-9780136915713

**Modern Software Engineering: Doing What Works to Build Better Software Faster**..
[https://www.informit.com/store/modern-software-engineering-doing-what-works-to-build-9780137314911](https://www.informit.com/store/modern-software-engineering-doing-what-works-to-build-9780137314911)


Perhaps one of them will convince you more than the other of the benefits of following a test-driven approach to developing software. But my hope is that you will be convinced, as I was, that test-driven development is the future of building a software engineering approach to building software. It is a fundamental technique that needs time and practice to do well. It improves everything one does when developing software. 

My pledge to myself is to adopt a more test-driven development approach. To develop the skills necessary so that one day I can refer to myself as as software engineer.
