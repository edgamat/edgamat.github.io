---
layout: post
title: 'The Theory of Agile Refactoring'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I feel refactoring my code is one of the best ways to maintain a well-designed, readable codebase. I also think helps the most when you perform the refactorings while you are developing your code, not after. 

<!--more-->

I was in my late 20s when the Agile Alliance gain a lot of attention. I read as much as I could including the books on Extreme Programming and one of its principles called Refactoring. I have my original copy of "[Refactoring, Improving the Design of Existing Code][book]", by Martin Fowler and it still resonates with me after all this time.

Within its pages you'll find these definitions of refactoring:

_**Refactoring** (noun)_: a change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior. 

_**Refactoring** (verb)_: to restructure software by applying a series of refactorings without changing its observable behavior

As Fowler points out, at some level, refactoring your code is about cleaning it up. But it is so much more as well. Refactoring is about being efficient and productive while you clean things up. It is a skill you learn and something that you need to work at in order to do well.

### My Context

I am hesitant to say that _anything_ is a best practice. It is very rare that any technique, principle or practice is universally the _best_ possible thing you can do. I know a lot of people get hung up on this sort of thing, and I am guilty of it as well. But the reality is that everyone's context is different. What might work amazingly well for one person or team might not work as well for another. And that's okay. Really. 

So is refactoring your code a 'best practice'? The answer is not always. It only makes sense if you have a supporting set of unit tests. I have tried to refactoring code that doesn't have supporting unit tests. I usually end up regretting it. Without supporting tests I usually will introduce one or more bugs when I refactor my code. If you want to refactor your code, then having a good set of unit tests is a very important.

I have, for the past few years, worked on projects following one sort of agile methodology. Currently, the team I work on follows the Scrum methodology. We work in two week sprints, peer review each code change, and demo the changes to the stakeholders at the end of the sprint. We  then schedule the changes for deployment into production. As a result, users expect to have new features, corrections or patches every couple of weeks. To achieve this cadence, our team follows a number of 'best practices':

- Trunk-based development
- Unit tests supporting each change to the code
- Use pull requests to peer review each set of changes
- End-to-end integration tests (both manual and automated)
- Automated builds and deployments (CI/CD)

In such an environment, we have all the ingredients for safely refactoring the code. We have the 'why' and the 'how' taken care of. How about the 'when'?

### When to Refactor Your Code

In Fowler's book, he covers this briefly. He opposes the idea that you should set aside time for refactoring. The sample he gives is allocating two weeks every couple of months to do refactoring. He rejects that idea because it doesn't match his view of refactoring. Refactoring should be done "...all the time in little bursts".

He then describes in more detail three times to refactor:

- Refactor when you add a function
- Refactor when you need to fix a bug
- Refactor as you do a code review

I agree with all of these approaches. I have used each one of these and can say that they have all made me improve the code's design as I performed them.

These are all activities you do "in the moment", not something you schedule to do later. And this is where my theory comes in.

### The Theory of Agile Refactoring

It goes like this:

_Good refactoring ideas from your current Sprint are less likely to be acted upon in the next Sprint_

Often the following conversation will happen between me and a team member:

_"This code is hard to understand in places and it looks like we've introduced some duplication"_

_"Should I spend the time now and refactor the code?"_

_"Let's get the current code merged into the main branch and in the hands of the test group. Then we can worry about refactoring the code."_

This thinking is the first step away from Fowler's notion of refactoring. I worked with one Tech Lead who spotted potential refactorings in the code and would recommend them be done before approving pull requests. And I've worked with others that did not. It really depends on the team and the type of work being done. But I tend to lean towards the same ideal. We really should be doing refactorings no later than the code review stage of a set of changes.

But let's say we don't. The reality is that sometimes we don't spot a potential refactoring opportunity until after the work is committed to the main branch of the codebase. Or we may agree (as the conversation above describes) that it is more important to get the code in front of the testers while we work on refactoring some of it. And when that occurs, we have to think about when to perform the refactoring work.

In my experience, if you decide to separate out the code changes and refactoring work, the refactoring work has less of a chance of being performed. This is the concept of _technical debt_ where the code manifests problems that eventually need to be fixed in order to maintain forward momentum. 

Refactoring is much more effective when done as you write your code. It is less effective if you do it after some time has passed. And it become less and less effective the longer you wait. The Theory of Agile Refactoring recognizes that if you wait until after the current Sprint is done to schedule a code refactoring, you are probably never going to actually do it.

### Reality for Most Teams

In my experience most teams have the best of intentions to refactor the code they write. But the reality is that if they wait to schedule it for later, it never gets done. I've tried to put refactoring activities on my "To Do List", or even put 'technical' stories in the backlog of a project's work stream. But they rarely result in the refactoring being done. 

The moment you allow your thoughts to view refactoring as something that can be done later, you've admitted it most likely will never happen. 

The Theory of Agile Refactoring recognizes this reality and suggests that teams should strive to complete refactorings before the end of the Sprint they are currently working on. That is the longest amount of time you can wait to refactor your code. If you expect to get the refactoring done in the next sprint, you probably won't. 

My (non-scientific) observations make the probability of completing a refactoring:

<table style="width: auto">
    <tr><th>While writing the code:</th><td>100%</td></tr>
    <tr><th>As part of a code review:</th><td>95%</td></tr>
    <tr><th>As part of fixing a bug:</th><td>85%</td></tr>
    <tr><th>As part of adding a function:</th><td>75%</td></tr>
    <tr><th>Before the end of the Sprint:</th><td>50%</td></tr>
    <tr><th>After the end of the Sprint:</th><td>25%</td></tr>
    <tr><th>After the end of the the next Sprint:</th><td>0%</td></tr>
</table>

### Summary

The Theory of Agile Refactoring is one that I have ample (anecdotal) evidence to support. It theorizes that the longer you wait to perform a refactoring, the less effective it will be and less likely it will occur at all. So don't wait!


[book]: https://refactoring.com/
