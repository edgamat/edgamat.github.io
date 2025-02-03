---
layout: post
title: 'Code Change Guidelines'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Here are the guidelines I try to follow when I make changes to a code base.

<!--more-->

## The Context

These guidelines make the most sense when the following conditions are true:

1. You are using `git` as the version control system,
2. You are working with a team of other developers that are also making changes to the code,
3. Your code changes are made in feature branches and shared with your team before merging them into
the main code branch, and
4. You expect the software to grow and evolve over time.

Given this as the context, these guidelines should apply to just about every code base I
work on as a professional software developer.

I didn't have the best of introductions to `git`. It was through an extension to Visual Studio. The
extension didn't use `git` native concepts. Instead, the extension mapped the `git` concepts into
Team Foundation Services (TFS) Version Control concepts. Using TFS, you had to 'check out' the files
you wanted to change, and then 'check in' the changes. Initially, these terms were used in Visual
Studio to make adopting `git` less confusing for TFS users. But at least in my case, this made
things more difficult in the long run.

Over time I was exposed to the strengths of the `git` model of code changes. I cannot imagine working
with code as I used to.

These guidelines are meant to help each developer communicate their changes to the rest of the team.
More often that not, developers work by themselves when making code changes. The first time another
developer sees the changes can be confusing without proper care being taken by the author of the
changes.

As software evolves, it can be difficult to understand when and why changes occur. These guidelines
can help you tell a story with your changes. Properly made changes can make it easy to understand
when a specific change was made, and why it was made. The ability to read and understand the code
is critical to the sustainable growth of your code base. In addition, it can save you a lot of time
when problems occur and you need to fix issues that are time-sensitive.

### Is a code change a single commit or a set of commits?

These guidelines can apply to either a single commit or to a set of commits. A set of commits, in
the form of a pull request, should follow these guidelines. But so too should the individual
commits. The relative importance you and your team give to each commit in a pull request can vary
depending on many factors. But I would recommend you start applying these guidelines at the individual
commit level first and move up to the pull request level once you are used to them.

## The Guidelines

- Each code change should include a message describing the change
- Each code change should be for a single purpose
- Each code change should be a reasonable size
- Each code change should be revertible

Adopting these guidelines will likely mean that you have to change how you think about code changes.
I know it did for me. It forced me to be more intentional about the changes I made. It required me
to break problems apart into smaller pieces. It required me to re-write the changes I had made in
order to tell a story.

### Each code change should include a message describing the change

I can remember the first time I was asked to use `git` as a version control system. The thing that
confused me the most was the requirement to include a message describing your changes. I could see
doing it on some changes, but all of them? Man, that just seemed crazy. It seemed crazy because I
couldn't imagine how to work in a way that made that possible. Not only is it possible, it is an
important tool in letting your code changes tell their story.

If you search the Internet you will find a lot of articles on how to write good commit messages.
Conventional Commits ([https://www.conventionalcommits.org/](https://www.conventionalcommits.org/))
is a common practice. I recommend you (and your team) document your set of practices for commit
messages. Regardless, it is important to understand that a commit message is more than just a short
description of the changes. A commit message can also include a body where you can describe more
about the commit, especially the 'why' behind it. The message should also include references to the
work item that inspired the work (like the User Story # in your work tracking tool).

Spending time crafting meaningful messages can be challenging. But I think it is worth the effort. If
you cannot describe the commit using a short message, then maybe your commit needs to be re-thought.
Maybe you have made the commit responsible for too many things. If you cannot describe why you are
making the changes, maybe you shouldn't be making them in the first place.

### Each code change should be for a single purpose

Whether a single commit or a set of commits, a code change should have a single purpose. This means
you should recognize different purposes and split them apart into separate code changes. A term used
to describe this approach is "atomic commits". Each commit contains a single unit of work. Here are
some examples of changes that should be in separate code changes:

- Implementing a new feature - this keeps the work separate from other unrelated changes
- Fix a bug - easier to revert, or tack down if the problem persists
- Refactoring - ensures refactorings are kept separate and will be easier to review
- Dependency updates - helps trace breaking changes when upgrading to a new framework version or 3rd
party library.

Creating separate code changes for these items elevates their visibility in the commit history. Their
part of the story is made more important, making it easier to understand why the code has changed.

### Each code change should be a reasonable size

The larger a code change is, the more challenges you will face. It is more difficult to review a
large set of changes. This slows down your team's ability to complete work. It is more difficult to
revert a large code change when it contains a bug.

Each team needs to decide what is a 'reasonable size'. It depends on a lot of factors. Does the number
of files changed make a code change 'too large'? Is it the number of lines of code that have changed
that makes a commit unreasonable? Use your best judgement, but lean more into the opinions of the
ones reviewing the code, rather than the ones authoring the code.

### Each code change should be revertible

Every commit should be a standalone, build-able snapshot of the codebase that passes all tests and
runs correctly. In other words, if you check out any commit, the code should compile, all unit tests
should succeed, and the application should function as expected. This ensures that each commit can
be reverted without breaking other functionality or introducing inconsistencies.

Taking the time to ensure each commit embodies these characteristics requires more effort on the part
of the author of the changes. t can also mean that more time needs to be taken rebasing and squashing
commits in order to keep each one self contained and revertible. But I feel this extra work is
worthwhile. It means the team can spend more time on intentional changes and less time on breaking
code and messy corrections. The team can feel more confident in rolling back changes without
unrelated harm. And it produces a cleaner history making it easier for onboarding new developers and
understanding the code's evolution.

## Summary

These code change guidelines are not rules to be followed but rather approaches to be embraced. It
requires each developer to think about the code they are changing differently. Coding is more than
making the code behave a certain way. It is also about telling a story for you and your team mates.
The story your code tells helps you sustain its growth as it evolves.

## References / Recommended Reading

Looks Good To Me: Constructive code reviews  
<https://www.manning.com/books/looks-good-to-me>

Conventional Commits  
<https://www.conventionalcommits.org/>

The Power of Atomic Commits in Git  
<https://dev.to/this-is-learning/the-power-of-atomic-commits-in-git-how-and-why-to-do-it-54mn>

Atomic Commits  
<https://en.wikipedia.org/wiki/Atomic_commit#Atomic_commit_convention>
