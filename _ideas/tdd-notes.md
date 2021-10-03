## The Three Laws of TDD

The essence of TDD entails the discipline to do the following:
1. Create a test suite that enables refactoring and is trusted to the extent that
passage implies deployability. That is, if the test suite passes, the system
can be deployed.
2. Create production code that is decoupled enough to be testable and
refactorable.
3. Create an extremely short-cycle feedback loop that maintains the task of
writing programs with a stable rhythm and productivity.
4. Create tests and production code that are sufficiently decoupled from each
other so as to allow convenient maintenance of both, without the
impediment of replicating changes between the two.

### The First Law
Write no production code until you have first written a test that fails due to the
lack of that production code.

### The Second Law
Write no more of a test than is sufficient to fail or fail to compile. Resolve the failure
by writing some production code.

### The Third Law
Write no more production code than will resolve the currently failing test. Once
the test passes, write more test code.

### The Fourth Law *
First you write a small amount of failing test code. Then you
write a small amount of passing production code. Then you clean up the
mess you just made.


## Test-Driven Development Rules

Rule 1: Write the test that forces you to write the code you already know you want to write.

Rule 2: Make it fail. Make it pass. Clean it up.

Rule 3: Don’t go for the gold.

Rule 4: Write the simplest, most specific, most degenerate* test that will fail.

(* The word degenerate is used here to mean the most absurdly simple starting point.)

Rule 5: Generalize where possible.

	generalization mantra => As the tests get more specific, the code gets more generic. (See Rule 13)
	
Stairstep tests: Some tests are written just to force us to create classes or functions
or other structures that we’re going to need. Sometimes these tests are so degenerate
that they assert nothing. Other times they assert something very naive. Often
these tests are superseded by more comprehensive tests later and can be safely
deleted. We call these kinds of tests stairstep tests because they are like stairs that
allow us to incrementally increase the complexity to the appropriate level.	

Misplaced responsibility: A design flaw in which the function that claims to perform
a computation does not actually perform the computation. The computation
is performed elsewhere.

Rule 6: When the code feels wrong, fix the design before proceeding.

Refactoring: A change to the structure of the code that has no effect upon the
behavior.

Rule 7: Exhaust the current simpler case before testing the next more complex
case.

It often happens, to TDD novices, that they find themselves in a pickle. They
write a perfectly good test and then discover that the only way to make that
test pass is to implement the entire algorithm at once. I call this “getting
stuck.”

Rule 8: If you must implement too much to get the current test to pass, delete that
test and write a simpler test that you can more easily pass.

Rule 9: Follow a deliberate and incremental pattern that covers the test space.

Rule 10: Don’t include things in your tests that your tests don’t need.

Rule 11: Don’t use production data in your tests.

### The TDD Uncertainty Principle

The TDD uncertainty principle: To the extent you demand certainty, your tests
will be inflexible. To the extent you demand flexible tests, you will have diminished
certainty.

Rule 12: Decouple the structure of your tests from the structure of the production
code.

Rule 13: As the tests get more specific, the code gets more generic.


Rule 14: If one transformation leads you to a suboptimal solution, try a different
transformation.


