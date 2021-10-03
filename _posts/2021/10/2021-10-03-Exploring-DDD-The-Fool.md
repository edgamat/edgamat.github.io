---
layout: post
title: 'Exploring Test-Driven Development - Fooling Yourself'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Test-Driven Development, is a fascinating topic. Robert C. Martin heralds it as a foundational discipline of software development. Others, aren't so convinced. Even though I have written my thoughts on this topic once or twice over the years, I was inspired by Martin's recently released "Clean Craftsmanship" to explore it further. This is the first exploration, focused on how you can fool yourself into thinking your way is better.

<!--more-->

### What is Test-Driven Development? 

Test-Driven Development, or TDD, is a discipline used to write software. At its core, the goal is to use automated unit tests to guide your work. You write a unit test that you know will fail, add just enough code to make the test pass, and refactor your code to incorporate the new code properly. Using test cases to guide your work can be very challenging for newcomers. I still struggle with it after 20 plus years of use. 

I'll let more experienced and qualified people try to teach and advocate TDD. My goal here is to look at the challenges I have faced when using TDD and explore ways of writing better software with TDD.

### Your First Test

I want to explore one of the domain behaviors relating to hashing file data. My application needs to know if the contents of a file have changed. I start with a failing test. I know (think??) I need a class to calculate the hash of a stream. So my test case wants to call a 'CalculateHash' function on a HashAlgorithm class:

```csharp
public class HashAlgorithmTests
{
    [Fact]
    public void CalculateHashReturnsValue()
    {
        var algorithm = new HashAlgorithm();
    }
}
```

This fails to compile because the `HashAlgorithm` class does not exist. So I create it. Now the test passes.

Next, I want to call the `CalculateHash` method:

```csharp
[Fact]
public void CalculateHashReturnsValue()
{
    var algorithm = new HashAlgorithm();

    using var stream = new System.IO.MemoryStream();

    var hash = algorithm.CalculateHash(stream);
}
```

This fails too, because The `CalculateHash` method does not exist. So we add it:

```csharp
public class HashAlgorithm
{
    public string CalculateHash(Stream stream)
    {
        return "";
    }
}
```

And the test case passes. But we haven't done anything meaningful with the hashed value in the test. Let's change that:

```csharp
var hash = algorithm.CalculateHash(stream);

Assert.True(hash?.Length > 0);
```

This fails, so let's add some 'real' code. I know from experience, an MD5 hash is where I want to end up, so I add this code:

```csharp
public string CalculateHash(Stream stream)
{
    using var hash = MD5.Create();

    return Convert.ToBase64String(hash.ComputeHash(stream));
}
```

And my test passes. 

### Sometimes, Things Don't Fail

I added the previous code using the MD5 hash because I have used this sort of code before and 'knew' what the 'correct' solution needed to be.

However, from a TDD perspective, that might not have been a wise thing. I've added code that makes it difficult to write a failing test.

The real test of this `CalculateHash` function is ensuring that it returns the same hashed value for the same stream of bytes. It is pretty easy to do that, even with hard-coded values (e.g. return `stream.Length.ToString()`). But if I wanted to be strictly following TDD, I'd have to write a series of failing tests to get me there. Perhaps, something like this would have been a good next test to write?

```csharp
[Fact]
public void SameCalculateHash()
{
    var algorithm = new HashAlgorithm();

    var contents = "Hello, World!";
    var contentBytes = Encoding.Default.GetBytes(contents);

    using var stream1 = new System.IO.MemoryStream(contentBytes);
    using var stream2 = new System.IO.MemoryStream(contentBytes);

    var hash1 = algorithm.CalculateHash(stream1);
    var hash2 = algorithm.CalculateHash(stream2);

    Assert.Equal(hash1, hash2);
}
```

With my new code in place, this new test passes. So no failing test. But if you look closely this new test doesn't tell me the code is correct. It can't tell the difference between these two implementations:

```csharp
public string CalculateHash(Stream stream)
{
    return stream.Length.ToString();
}
```

```csharp
public string CalculateHash(Stream stream)
{
    using var hash = MD5.Create();

    return Convert.ToBase64String(hash.ComputeHash(stream));
}
```

Perhaps my test case should have been this?

```csharp
[Fact]
public void SameCalculateHash()
{
    var algorithm = new HashAlgorithm();

    var contents = "Hello, World!";
    var contentBytes = Encoding.Default.GetBytes(contents);

    using var stream1 = new System.IO.MemoryStream(contentBytes);
    using var stream2 = new System.IO.MemoryStream(contentBytes);

    var hash1 = algorithm.CalculateHash(stream1);
    var hash2 = algorithm.CalculateHash(stream2);

    Assert.Equal("ZajifYh5KDgxtmS9i38K1A==", hash1);
    Assert.Equal("ZajifYh5KDgxtmS9i38K1A==", hash2);
}
```

But even this one passes. The 'trouble' I am facing is that I have leaped ahead to a 'correct' solution without having a failing test. Is that bad? Am I doing TDD wrong?

_(I'll admit here that this example is very simple... but imagine using TDD on a more complex problem that isn't as clear to everyone of what the 'correct' solution looks like)_

### The Trap

I feel like this is a trap I sometimes fall into when practicing TDD. I get to a point in the code where I can see a proper solution and skip to the end. This means my test cases didn't drive me there. And as a result, I probably will be faced with a situation where I can't easily write tests that fails. But the code 'feels' correct. I can write a bunch of tests that pass against my current code, but I can't write any that fail. Is that so bad?

On the one hand, I could justify my approach against following TDD in these cases. My ability to quickly see 'correct' solutions in the code is what sets me apart from other developers. Years of experience of making mistakes, learning from them, and improving my skills means I can sometimes design solutions quickly because I've seen the patterns before. Why should I use TDD to drive me towards a solution when I already know the destination? "My way" feels like the right thing to do and TDD is disrupting my 'flow'.

But on the other hand, it's a trap you can set for yourself. My 'correct' solution may only be one of many possibilities. I didn't get a chance to explore them through tests. I might have arrived at a completely different (and possible better) solution. What if I had been programming collaboratively with another developer? They may not have been able to follow my 'leap' to the end solution. To them, my solution might be confusing.

This 'trap' is common in a lot of scenarios. It's your own bias for a particular outcome that makes you find ways to justify it. In the case of code, when you leap forward to an end solution, you can't write any failing tests, so you write a bunch of passing tests, demonstrating the correct solution. But that is not TDD. You let your bias for the solution cloud your ability to use evidence (ie failing tests) to get you there.

This approach to problem solving is called 'motivated reasoning'. You know something is true (your coded solution), so you find evidence to support that claim (you write a bunch of passing tests). Along the way you may miss a test that would demonstrate your code is wrong. Or even worse you may think of a test and not bother to write it (because it can't possibly be true if your solution is correct).

### The Lesson

Your instincts for a 'correct' solution can cause you to skip ahead and not follow TDD. Some developers might prefer this approach. It can be satisfying to have an "A-ha!" moment and code an elegant solution quickly. And you can fill in a bunch of test that all pass, which easily demonstrates your solution to be 'correct'.

But I would humbly suggest that you are fooling yourself into thinking that your way is better than practicing TDD. No matter how smart you may be, you can always be fooled into thinking your solutions are correct. And without following TDD, your tests will be written from that perspective. 

We typically give optimistic estimates to effort and timelines. We often submit work to be tested that doesn't satisfy all the requirements. We often make changes that cause a bunch of other parts of the system to fail. And this is due, in the most part, to fooling ourselves. 

My advice is to keep practicing test-driven development. When you have that "A-ha!" moment, or you feel strongly about a particular solution, it can be a good thing, if channeled through TDD. Use failing tests to be your guide. 


