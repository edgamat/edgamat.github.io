---
layout: post
title: 'Unit Testing Presentation'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
published: false
---
 

## Better Unit Testing - Lessons Learned

We spend a lot of time writing unit tests for projects. It is important to do a good job or else it is a lot of wasted effort.

I have been fortunate to work on a project that prioritizes unit testing and I have learned some valuable lessons.

I'd like to share these lessons with you so that you too can benefit from the experiences I had learning to write better unit tests.

I hope it will inspire you to create more valuable unit tests for your projects 

<!--more-->

## What is the Goal of Unit Testing?

> The goal is to enable **_sustainable_** growth of the software project

![Test Goals](/assets/img/test-goals.png)

Without careful attention, over time the amount of disorder in the software will grow. With more disorder, you will spend more time fixing problems than making progress:

- Fixing one bug will cause one or more to appear. 
- Changing one piece will cause another piece to break.

Creating unit tests will guard against these issues making it easier to make changes as the size and complexity of the system increases.

## Concepts

A **unit test** is an automated test that
- Verifies a small piece of code (also known as a **unit**),
- Does it quickly,
- And does it in an isolated manner.

A 'small piece of code' may represent a function, a property, an entire class, or a small set of classes. It depends entirely on the context. 

### Test Doubles - Stub vs. Mock

A **test double** is a piece of code used to replace
a class's dependencies when executing unit tests. Test doubles can
help make tests easier to maintain and set up. 

A **stub** is a test double that fakes the behavior of a dependency.  
A **mock** is a test double that a unit test will inspect to determine
if a test was successful.

Examples:  
- The system under test (SUT) retrieves data from a test double in order to execute
the test. This test double is a **stub**.
- The SUT changes the state of a test double. The test then inspects
the test double (with assertions) to confirm the correct behavior
occurred. This test double is a **mock**.

NOTE: When testing a method on a class, it is not required that all class dependencies be test doubles. It is acceptable for more than one 'real' class be used in a unit test. Granted this is rarely necessary, but don't feel like you must create test doubles for everything. It can lead to a lot of un-necessary code.

Case Study:

AutoMapper is a C# library that can convert one class to another using a set of transformation rules. We used AutoMapper on a project to map incoming request payloads to domain objects. In our test classes, we used test stubs to simulate the AutoMapper `Map` functions. This made it possible to avoid using AutoMapper directly in our tests. We felt it was difficult to configure AutoMapper in our tests and it seemed to run slowly as well.

All seemed fine but more and more a similar pattern would occur. We'd have all our tests passing, but the system would crash when deployed to our integration environment. The errors were due to changes in the rules used to transform the classes. If we changed the rules, we didn't go into all the unit tests and update our test stubs to match the new behavior. 

So tests would still pass, even though the code would fail when actually put to use. These are 'false negatives' and caused us a  lot of grief. 

### False Positives and False Negatives

A **false positive** is a false alarm. The code still behaves correctly, but a change in the code has caused a test to fail.

A **false negative** is when a change in the code has introduced a bug but the test(s) still pass.

<font color="#f00">Demo</font>


## What is Good Unit Test?

A good unit test has the following four attributes:
- Protection against regressions
- Resistance to refactoring
- Fast feedback
- Maintainability

### Protection against regressions

- A _regression_ is a software bug
- When a feature stops working after a modification to the code
- Maximize by having tests exercise as much code as possible

### Resistance to refactoring

- Tests don't fail when the underlying code is refactored (false positives)
- Don't depend on implementation details
- Verify the end result, not the steps to get there

![Test Relationships](/assets/img/test-relationships.png)

Source: Figure 4.3
_Unit Testing: Principles, Practices, and Patterns_

### Fast feedback & Maintainability

- Run tests quickly
- Write maintainable tests:
    - easy to understand
    - easy to run

## Lessons Learned Coding
 
### Helper Functions
<font color="#f00">Demo</font>

### Refactor, Refactor, Refactor
<font color="#f00">Demo</font>

### Fluent Assertions
<font color="#f00">Demo</font>

### Focus on Behavior, Be Explicit
<font color="#f00">Demo</font>

### Testing Database Calls
<font color="#f00">Demo</font>

## Testing HTTP Calls
<font color="#f00">Demo</font>

## Lessons Learned Designing

### Use Automated End-2-End Tests
<font color="#f00">Demo</font>

### Use Code Coverage Tools
<font color="#f00">Demo</font>

- NCover, dotCover
- Easy to spot areas of the code lacking unit tests
- Don't focus on percentages too much (at first)

## Is it worth it?

- Having a good coverage of automated tests is worth the effort
- You have to include testing with all your estimates
- After a few iterations, you won't be able to separate coding
  time from testing time.

## Practice Makes Perfect

- Make a small set of changes
- Confirm the new behavior with a reasonable set of unit tests
- Share your code with others






A powerful take away from reading your book for me was the idea that how the code is designed has a significant impact on the unit tests written to support that code. So I have to ask the question, “Are the mocks I am using in my tests necessary, or a side-affect of how I have written the code?”

On a current project, I tried an experiment. I took a set of unit tests and removed all test doubles except the single out-of-process dependency (an EF Core DbContext class). I immediately discovered a good number of the tests no longer worked. This was due to how often the test were verifying implementation details (using the “Verify” function with Moq) which would no longer work with the real classes. 

So I refactored all the tests to stick to only asserting on observable behavior. That too caused other problems since some tests could not observe the behavior directly. So I questioned if these tests were necessary at all. 

I found that several of them were indeed necessary. I refactored the domain classes to be more functional in nature with fewer side effects. This made the unit tests much easier to write.  

 

I think this is very true when seeing folks adopt a London or Classical approach.

As you pointed out, using mocks to avoid creating a large object graph is probably a warning side that the code isn’t designed as well as it should be. 

And I must agree with your assessment; if your domain model is overly complex due to header interfaces, its probably time to refactor to a simpler design. The resulting unit tests will be of higher quality because the underlying code is as well.
