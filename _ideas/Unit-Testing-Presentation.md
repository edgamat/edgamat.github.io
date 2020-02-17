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

A test double is a piece of code used to replace
a class's dependencies when executing unit tests. Test doubles can
help make tests easier to maintain and set up. 

A stub is a test double that fakes the behavior of a dependency.
A mock is a test double that a unit test will inspect to determine
if a test was successful.

Examples:
The SUT retrieves data from a test double in order to execute
the test. This test double is a stub.

The SUT changes the state of a test double. The test then inspects
the test double (with assertions) to confirm the correct behavior
occurred. This test double is a mock.

Demo

Repository -> stub
Logger -> mock

### False Positives and False Negatives

Demo

## What is Good Unit Test?

### Protect against regressions

### Resistance to refactoring

### Fast feedback

### Maintainability

## Lessons Learned

### Helper Functions

### Refactor, Refactor, Refactor

### Fluent Assertions

### Focus on Behavior

### Avoid using Collaborators

## Is it worth it?

- Make a small set of changes
- Confirm the new behavior with a reasonable set of unit tests
- Share your code with others

