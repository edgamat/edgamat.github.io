---
layout: post
title: 'Do I Write Good Tests?'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When I write code, I always try to include unit tests. Sometimes I feel like they are good tests, and
other times, I feel like I could have done better. Do I write good unit tests?

<!--more-->

How does one evaluate a good unit test? In the book "[The Art of Unit Testing][art]", Roy Osherove writes:

> The tests that you write should have three properties that together make them good:
>
> - **Trustworthiness** — Developers will want to run trustworthy tests, and they’ll accept the test results with
>   confidence. Trustworthy tests don’t have bugs, and they test the right things.
> - **Maintainability** — Unmaintainable tests are nightmares because they can ruin project schedules, or they may be
>   sidelined when the project is put on a more aggressive schedule. Developers will simply stop maintaining and
>   fixing tests that take too long to change or that need to change very often on very minor production code changes.
> - **Readability** — This means not just being able to read a test but also figuring out the problem if the test seems to be wrong. Without readability, the other two pillars fall pretty quickly. Maintaining tests becomes harder, and you can’t
>   trust them anymore because you don’t understand them.

As a comparison, in the book "[Unit Testing: Principles, Practices, and Patterns][ppp]", Vladimir Khorikov writes:

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

My primary motivation for writing
any unit test is protection against regressions. I don't (typically) focus on proving my code
works correctly. Answering that question is something more involved than writing a few tests.
That's not to say I don't discover bugs while writing unit tests. I usually do. But my primary focus is
about future changes to the code. I want to leave behind protections for me and the other developers. I
hate introducing bugs when making changes. So how successful at it am I?

## Approach

I don't practice Test-Driven Development (TDD) as often as I'd like. I recognize the power of this
practice and its positive effect on code. However, I fail to instinctively use TDD first. My first
instinct is to write a few pieces of code, then write a test or two to verify the code I've written.
It leaves me with a good coverage of tests, but I always have felt I could be doing better.

I don't like writing tests using the implementation details as a guide. It makes me feel like I'm
'going through the motions', or writing tests because I have to. I'd much rather look at the code from
the perspective of behavior. I like tests that answer the question "What does the user expect
this code to do?" Khorikov and others describe this as testing the _behavior_ of the code. I like
that way of thinking. Test the behavior of the public API, not the details. Having said that, it's
admittedly a work in progress for me. I still write too many tests based on implementation details.

## Speed

I've never had issue with the speed at which the tests run. The feedback loop is always fast enough.
When I first started writing unit tests, I used them to bypass testing via the user interface (UI). It
was so much quicker to write a unit test and be able to run it over and over to check my code. Having
to navigate through the UI and test that way was wasteful and slow. But just having the unit tests in
place meant I could run tests quickly and see how changes to the code affect the behavior.

## Refactoring

How do my tests hold up to refactorings? I'd say okay, but with a lot of room for improvement. When I
make a refactoring, and I see a test fail, I'm usually pleased. It means I've covered some code with some
test and it demonstrated that the changes impacted the behavior. In theory, this just caught a regression.
However when I investigate the cause of the broken test, it sometimes isn't because of the refactoring. It
is because the test was tied to an implementation detail, not an expected behavior. I changed the implementation,
but didn't make any changes to the expected behavior. This is the sort of thing I need to avoid because this
broken test is really a _false positive_.

## Using Test Doubles

The question is how to achieve proper test coverage, without basing the tests on how things are implemented.
I always thought this was more difficult to achieve than it might be. Recently in Khorikov's book on unit
testing, he advocated two schools of thought on approaching unit tests: London and Classical.

- The first, the "London" approach, advocates isolating the piece of code unit test. In practice, this results in
  using a lot of test doubles to isolate the code so it can be tested without any outside influences.

- The second, the "Classical" approach, advocates isolating each unit test, in order for the the running of
  each test doesn't influence others.

Reading through Khorikov's descriptions of these two approaches and the benefits and drawbacks, I could see
these patterns in my own code. My tests where much more focused on isolating the code, rather than isolating
the tests. My general feeling was that isolating the tests (using test doubles) meant the tests would run
faster, and I could be better protection against regressions. So I'd definitely represent a practitioner of
the "London" approach.

However, when I looked at my tests I noticed two things. First, I was writing a significant amount of code
creating test doubles (a liability). It definitely made the tests more difficult to maintain and read.
Second, I would get a number of tests that would _NOT_ fail when I made a change in the code, when I expected
them to. It was because I had too many test doubles in place and they didn't care that the real code had changed.

One example would be mapping logic. We use AutoMapper to help map data contract models (the JSON received from
calling programs) to domain models in REST API endpoints. We found that using AutoMapper was difficult
and sometimes too slow so we started using test doubles for the mapping functions. This sped up running
the tests and we kept using this approach for new functionality. However we started encountering problems when
changes were made to the mapping functions. Without the changes being reflected in the test doubles, the tests would fail. More and more we'd require the full mapping logic to be replicated in the test double, making it harder to maintain
the test doubles over time.

## Making a Change

I decided to experiment and remove the test doubles as much as possible in a set of unit tests I was working
on. The tests covered a service class with about a half dozen tests. Running the tests required 5 test doubles;
four collaboration classes and an EF Core database context class (mocking the data access layer). I had
two situations where the tests did not fail when I changed the collaboration classes. As a result,
it wasn't until I ran an integration test that the issues were discovered.

Replacing the test doubles with the real classes immediately revealed the problem via failing unit tests.
It turns out that the test doubles were not updated to reflect the new behavior and that was why they didn't
report a regression issue by failing the unit tests they were used in.

Using the test doubles always appeared correct. I felt I would always know what part caused the test to fail
because only the system under test was in play. Everything else was a well-controlled simulation and it felt
right. However, as this example points out, things are sometimes more complex than you think.

Using the "London" approach wasn't going to help protect me against all types of regressions. So perhaps
using fewer test doubles was something to consider.

## A-Ha!

In the past I had seen posts from Kent Beck and others warning against the overuse of mock classes. It never
made an impact on me. But seeing the issue of using mocks hiding regression issues was an eye-opener for me.
It made me realize now what he and others had trying to explain. Using mock classes, or test doubles in general,
was a powerful technique in writing good unit tests, however one shouldn't go overboard and mock everything.

In addition to the issue I demonstrated in my example, preventing regressions from being discovered until
integration testing, over-use of test doubles also reduces the coverage of your unit tests. If I have a class
being tested and it in turn calls its collaborators, one test will be covering a portion of the code in the
collaborators as well. This increases the code coverage with very minimal effort. In theory, this should reduce the
amount of test cases necessary to provide sufficient code coverage.

## When to Use Test Doubles

Khorikov has some great observations in his Unit Testing book. One I'll paraphrase here is to attempt to
only use mocks for out-of-process dependencies. He doesn't come out explicitly and advocate this, but
he suggests this mindset is a better approach than mocking all class collaborators.

So, what would that mean if applied to my situation? I would treat the database access as an out-of-process
dependency, so creating a test double for the EF Core DbContext is reasonable. As for the other collaborators,
I shouldn't look to create test doubles for them, unless necessary. Keep using the real implementations until
using them becomes painful.

Besides the database access, what other out-of-process dependencies might I need to create test doubles
for? Here are a few I use frequently:

- Time. Anything that relies on the operating system (`DateTime.UtcNow`, `Task.Delay` for example) is a good candidate.
- Files. Access to the file system is something that will slow down the tests and make them brittle.
- Message Bus. We use MassTransit, a message bus for .NET. Creating test doubles for this functionality
  is critical to ensuring proper test isolation.
- HTTP. Any HTTP call needs a test double, just like accessing the database or file system.
- SMTP. Sending email messages via SMTP is also something that must be simulated during unit tests.

## Another Useful Observation

One other thing I felt peeking through all these testing questions was the code being tested. The code
I was testing was mostly based on an _[anemic domain model][anemic]_ using service classes for most business logic
and operations. The quality of the unit tests (aka the goodness of the tests) was directly related to
the quality of the code being tested. So if the tests feel like they aren't good, it is possible, in fact
likely, that the root cause of the issue is the code, not the tests.

Exploring improved designs and techniques will undoubtedly lead to better code and tests. Using Functional
programming techniques, employing a richer domain model, and organizing the dependencies according to
Clean Architecture principles will improve the quality of the unit tests.

## Slow and Steady

Getting back to my example, I started off by replacing the test doubles for one service class and had
positive results. Should I now retrofit that approach into every test in the system? I discussed this
with a co-worker. (By the way, having smart, experienced peers is better than any advice from any book...)
He recognized what I was wanting to do and why. But he asked me if the change in approach would work in all cases. Sometimes the mocked classes are hiding a huge amount of infrastructure that would be necessary to build in
order to run a test with the real collaborator. He also was concerned that additional effort supporting the
unit tests might be better focused on other areas, like improving test coverage.

We agreed that relying less on test doubles was a good thing, but let's start small and begin using this
new technique as we saw fit. It would allow us time to see it in action, weight the benefits and
drawbacks and get a feel for its effectiveness in our context.

## One Last Thought

That last part of the story is the critical part to remember: _context_. All the techniques and ideas
from TDD to the Classical/London approaches all have to be viewed from within your context. Anyone who
says TDD must be used at all times is not doing you any favors. Anyone that says the way you are
doing something is terrible and _their_ way is so much better is not your friend.

Only you can really know if a technique or approach is suited for your context. Don't brush off
new ideas just because they are different or seem impossible to grasp. However, don't let self-doubt
and fear grip you either because someone writes a book and tells you their way is _the Way_.

[art]: https://www.manning.com/books/the-art-of-unit-testing-second-edition
[ppp]: https://www.manning.com/books/unit-testing
[anemic]: https://en.wikipedia.org/wiki/Anemic_domain_model
