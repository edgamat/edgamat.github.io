---
layout: post
title: 'Unit Testing is Design Testing'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

This is going to be a short post that I put here to remind me in the future of an insight I read this past week. 

<!--more-->

Unit testing is an important skill to learn and greatly improves the quality of your code. But it might not be verifying the things you think it is.

### The Insight

According to [Vladimir Khorikov][unit-testing-book] when you write unit tests, the goal is to enable sustainable growth of your software. He states an important insight in his book "Unit Testing Principles, Practices, and Patterns", that there is a relationship between unit tests and code design:

> If you find that code is hard to unit test, it's a strong sign that the code needs improvement. The poor quality usually manifests itself in tight coupling, which means different pieces of production code are not decoupled from each other enough, and it's hard to test them separately.

When testing your code, you are testing the design. If a test is hard to write, your design is probably is to blame. Difficult to test? Bad design. That's it.

Test-Driven Development helps improve your design as you write the code. It provides you earlier feedback that your design needs improvement. But even if you wait to write tests until after your code is written, you can still use that experience to test your design.


[unit-testing-book]: https://www.manning.com/books/unit-testing
