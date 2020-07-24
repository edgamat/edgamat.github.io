---
layout: post
title: 'The Most Important Rule for Testing Existing Code'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When following Test-Driven Development (TDD), one creates a failing test and then writes code to make the test pass. But what if the code already exists?

<!--more-->

If a piece of code has no covering unit tests, it is important to add tests that demonstrate the desired behavior. The most important rule for adding tests to existing code is to _ensure you are testing the right thing_.

## Test the Right Thing

When I am adding tests for existing code, I usually write the test case with the knowledge of the observed behavior. So my tests pass immediately because I know how the code behaves. However this is a flawed testing strategy because these tests are biased towards examples that show how I _think_ the code behaves. 

It is common in the software development industry to hear people say that developers make poor testers of their own code. The reasons for this include our bias towards the 'positive' scenarios, and less towards how the code might break. We can be blind to parts of the code with flaws and it can have a great influence over how we write unit tests. 

In other words, this bias increases the chance that the unit tests aren't testing to right thing. To combat against this bias, I need to _confirm the test fails_.

## How to Confirm a Test Fails

The technique I use is to write the unit test based on the observed behavior. I can see the test passes (that 'positive' scenario again). Then I modify the existing behavior of the code in a way that _should_ cause the unit test to fail. When I re-run the test(s) and I don't see the expected failures, I know I have not tested the right thing.

I'm not going lie, it can be very disheartening to modify a piece of code that results in no failing tests when I expected it to fail. When this happens (and it does more often than I'd like to admit) it is a clear demonstration of the quality of your test suite. Or rather, the lack thereof. 

However, this confirmation of a test's ability to 'test the right thing' is a powerful thing. It amplifies my confidence that the test is adding value to the overall code quality. And when I don't see the test fail as I expect it to, correcting the test, or the code, is much easier since I am much more familiar with the current context than I would be days or weeks later when a bug is reported in the code. 

Speaking of bugs...

## Observation of Bugs

When a bug is reported, it is important to understand why the bug exists. Since I have a suite of automated unit tests for my code, my first question is to determine why didn't the bug get observed in an existing unit test? It boils down to one of two reasons. Either, there was insufficient test coverage and no tests were written to cover the code with the bug, or the test didn't test the right thing.

If there was insufficient test coverage to observe a bug in the code, the first step in correcting the bug is to write a test that should fail due to the bug. This TDD approach ensures that fixing the bug will be easier but also the bug will have a covering test in the future to protect against regressions.

If the bug slipped past our test suite because of a poorly written test, that is a much more interesting thing to look at. It means that I didn't understand the code's behavior as well as I thought I did. I could simply write a new test to observe the bug, just as I would when there was insufficient coverage. However, it is more important to look at the test I _expected_ to find the bug and understand why it didn't. 

Did I miss an assertion? Did I set up test doubles incorrectly? Did the test use a flawed set of inputs? Understanding what went wrong makes me a better writer of unit tests. It makes me more aware of what I need to do to test the right thing.

## Summary

Using a test-driven approach to designing software code has the advantage of watching tests fail, and ultimately confirming the desired behavior. But when you test existing code, you need to be more careful to ensure you are testing the right thing. The most important thing you can do is to see tests fail the way you expect them to. Combat your bias and test more than the 'positive' scenarios.

