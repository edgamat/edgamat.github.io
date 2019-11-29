---
layout: post
title: 'Do I Write Good Tests?'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When I write code, I always try to include unit tests. Sometimes I feel like they are good tests, and
other times, I feel like I could have done better. Do I write good unit tests?

<!--more-->

How does one evaluate a good unit test? In the book "The Art of Unit Testing", Roy Osherove writes:

> The tests that you write should have three properties that together make them good:
>
> - **Trustworthiness** — Developers will want to run trustworthy tests, and they’ll accept the test results with
>   confidence. Trustworthy tests don’t have bugs, and they test the right things.
> - **Maintainability** — Unmaintainable tests are nightmares because they can ruin project schedules, or they may be
>   sidelined when the project is put on a more aggressive schedule. Developers will simply stop maintaining and
>   fixing tests that take too long to change or that need to change very often on very minor production code changes.
> - **Readability** — This means not just being able to read a test but also figuring out the problem if the test seems to be wrong. Without readability, the other two pillars fall pretty quickly. Maintaining tests becomes harder, and you can’t
>   trust them anymore because you don’t understand them.

As a comparison, in the book "Unit Testing: Principles, Practices, and Patterns", Vladimir Khorikov writes:

> A good unit test has the following four attributes:
>
> - **Protection against regressions** — Exercise as much code as possible
> - **Resistance to refactoring** — The degree to which a test can sustain a refactoring of
>   the underlying application code without turning red (failing)
> - **Fast feedback** — Run quickly
> - **Maintainability** — Easy to understand, easy to run

One more thing that Khorikov mentions is also important to consider:

> An unfortunate fact of programming life is that **code is a not an asset, it’s a liability**.

Considering all this, I ask myself, "Do I Write Good Tests?" The short answer is I don't know. It is more
of a feeling I have, rather than any concrete metrics. I don't collect data on how well my tests
rate on the scales of maintainability, speed or protection against regressions.

## Motivation

I'll start by asking why I write tests. My primary motivation for writing
any unit test is protection against regressions. I don't (typically) focus on proving my code
works correctly. Answering that question is usually something more involved than writing a few tests.
That's not to say I don't discover bugs while writing unit tests. I usually do. But I'm usually thinking
about future changes to the code. I want to leave behind protections for me and the other developers. I
hate introducing bugs when making changes.
