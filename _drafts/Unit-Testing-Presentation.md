# Unit Testing Lessons Learned

## What is the Goal of Unit Testing?

> The goal is to enable **_sustainable_** growth of the software project

Without careful attention, over time the amount of disorder in the software will grow. With more disorder, you will spend more time fixing problems than making progress:

- Fixing one bug will cause one or more to appear. 
- Changing one piece will cause another piece to break.

Creating unit tests will guard against these issues making it easier to make changes as the size and complexity of the system increases.

## Concepts

A unit test is an automated test that
- Verifies a small piece of code (also known as a unit),
- Does it quickly,
- And does it in an isolated manner.

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

<font color="#f00">Demo</font>

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

<font color="#f00">Demo</font>

### Fast feedback & Maintainability

- Run tests quickly
- Write maintainable tests:
    - easy to understand
    - easy to run

## Lessons Learned
 
### Helper Functions
<font color="#f00">Demo</font>

### Refactor, Refactor, Refactor
<font color="#f00">Demo</font>

### Fluent Assertions
<font color="#f00">Demo</font>

### Focus on Behavior, Be Explicit
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

