---
layout: post
title: 'Eliminating Waste'
author: 'Matthew Edgar'
pub_date: '2019-11-10'
excerpt_separator: <!--more-->
---

The Lean Software Development Principles advocate the elimination of waste in your development
process. Let's take a look at what forms waste can take and how to reduce or remove it. Specifically,
when applied to .NET Core projects.

<!--more-->

## What is Lean Software Development?

Not surprisingly, Wikipedia has a good entry for defining [Lean Software Development][lean-wiki].
First introduced in the book [Lean Software Development][lean-book] by Mary Poppendieck and Tom Poppendieck in 2003. The book applies the principles of [lean manufacturing][lean-manufacturing] to the activities related to software development.

The first principle the book investigates is _Eliminate Waste_:

> "Waste is anything that does not add value to a product, value as perceived by the customer."

The authors contend that eliminating waste is the most fundamental lean principle.

Some examples of waste in software development are:

- Developed components not used by the customers
- Requirements gathered but not developed
- Doing more work than necessary
- Coding features not immediately needed
- Handing off work from one group to another

Reading this list, one can see that there may be different interpretations of what waste means. Ultimately
you will need to perform your own assessment in your own context. What is wasteful to you might not be the same
as another developer on your team, in your company or the industry as a whole.

The following are some (hopefully) useful ways of looking at eliminating waste when developing software.

## How to Recognize Waste

The first step in eliminating waste is to look around and decide what adds value to your customer. One
potential way to start this process is to separate activities, tools, facilities and processes into
the following classes:

- Value-adding: things that add value to your customer
- Non value-adding; but necessary: things that don't directly add value but are necessary to create value
- Non value-adding and not necessary: things that are pure waste that need to be removed

Let's take a look at how to see non value-adding aspects of software development. Our goal will be
to eliminate the parts that are not necessary and to reduce as much as possible the parts that
are still necessary to produce value.

### Partially Done Work

There are many forms of partially done work. For example, you create a branch of the codebase and begin working
on a feature. Until that code branch is merged into the master code base and deployed into production, it is
'partially done'. Let that sink in for a minute. Based on that definition, a significant portion of work can
potentially be 'partially done'. I'll contend that it is not possible to eliminate all of these activities. You
must have some amount of code 'in flight' in order to work the code changes in isolation, integrate them into
the master code base, have them tested and eventually deployed into production.

Keeping the amount of partially done work to a minimum is the key. [Agile software development][agile] methods help
in this regard. For example, having teams tackling small amounts of work (Sprints/Iterations) with the changes being
deployed to the production environment as soon as they are ready. Also, Continuous Integration [ci] is a
powerful technique for managing development. It reduces the time between making a change and having that change
in production (adding value to the customer).

### Extra Features

Software can become bloated over time. It can contain features never requested by the customers. It can
contain a lot of unnecessary complexity. It can be full of obsolete functions that no longer provide
value.

It is important to apply discipline when managing the code to eliminate these extra features. During development
you must ask yourself if a new feature is really necessary. Does it add value immediately or it is something
that might be deferred to a later stage of the development effort? It is critical to validate assumptions
throughout the development cycle. "Does this add value?" is the simplest and most powerful question to ask.

You can use a number of [refactoring techniques][refactoring] to help maintain a clean and well organized
code base. This is something that must be done continuously and diligently in order to be effective.

### Relearning

When working on a task, you learn things. This knowledge you gain should exist in the code you write
and the artifacts produced by your development process (user stories, task descriptions, bug reports).

It is wasteful to relearn this knowledge. This can manifest in several ways:

- You don't record important details about the tasks as you work them.
- You don't express the ideas clearly in the code.
- You don't share the knowledge with others.

On more than one occasion, I have been the direct benefactor (future self) of activities
I did (past self) to capture knowledge as I progress through the development effort. But sharing
that knowledge with others (collaboration, communication, clarity) is just as important.

### Task Switching

Switching between different tasks has never felt natural. It is a skill that I have had develop and
I constantly work at improving. Having to work on multiple projects at the same time is sometimes
a reality that cannot be avoided. However, keeping it to a minimum is going to help reduce the
waste that occurs when switching between tasks.

This also applies to my own development habits. If I am working on a feature, I should focus on it alone
and get it completed before tackling the next task. Getting into a flow that produces value is
what you want to achieve.

### Waiting

Waiting for things to happen can sometimes be a significant source of waste. All sorts of delays
are possible:

- Delays in starting a project/obtaining approval
- Delays in getting the right people to work on tasks
- Delays in getting features reviewed and merged into the master code base
- Delays in getting features tested/deployed

Each of these might not be significant by themselves. However when combined, they can provide
a major hinderance to having the customer realize value from your work.

### Handoffs

Each time work is handed off to another group or person, there can be waste. For example,
when you complete a task,

- How does the code get merged into the master code branch?
- Who deploys it into the shared environment for integration testing?
- How does it get moved into a staging environment for sign-off?
- Who deploys the changes into the production environment?

Automating these activities is one way of reducing waste. The most obvious of these
tools is to use a version control system (VCS) to manage your code. [Git][git] is the _de facto_
industry standard (as of late 2019) and I cannot recommend using any other VCS. Git
has literally transformed how the industry develops code.

[DevOps][devops] is a powerful set of practices and tools to deliver code changes quickly and reliably.
You can employ DevOps practices to automatically build code (including running unit tests) whenever code is
committed to the VCS. You can then automatically deploy the code to whatever environment you wish.
The sky is the limit as far as what can be automated. This freedom allows you to reduce the time
it takes to provide value to you customer.

### Defects

The amount of waste a bug/defect represents is directly correlated to when it is discovered. If
the defect is in the user story or requirement description, catching that early before the work
commences is the best case scenario. As you go through the development of the code, the cost
of correcting the issue becomes greater and greater, resulting in more and more waste.

Therefore finding errors as soon as possible is critical in eliminating waste:

- Unit testing can provide a great feedback mechanism for detecting issues as soon as they occur.
- Integrating your changes frequently, in small increments can make detecting issues simpler.
- Placing the new features in the hands of real users will let you know if there are problems. Do this as 
frequently as possible to reduce the time it takes to get feedback that something is wrong.

### Management Activities

Do management activities add value? No more than testing or refining your product backlog. But they
can be wasteful if not done correctly:

- Track your work efficiently. That can be with pen and paper, but the important thing is to not waste
  time trying to figure out what you're working on
- Ensure everyone is on the same page. Meet and discuss with your team. Share and communicate.
- When a task is blocked, know who to go to and get it unblocked.
- Let work flow through the development cycle as quickly as possible (this will eliminate a lot of management tracking)
- When you need approval for a task or a change in a task, know how to get that done.

## Summary

Learning to see waste is the first step in eliminating it from your software development process. Once you can identify
it, reduce it or make it go away completely. Strive to add value to your customers for the benefit of all.

[lean-wiki]: https://en.wikipedia.org/wiki/Lean_software_development
[lean-book]: https://www.amazon.com/Lean-Software-Development-Agile-Toolkit/dp/0321150783
[lean-manufacturing]: https://en.wikipedia.org/wiki/Lean_manufacturing
[agile]: https://en.wikipedia.org/wiki/Agile_software_development
[ci]: https://en.wikipedia.org/wiki/Continuous_integration
[refactoring]: https://en.wikipedia.org/wiki/Code_refactoring
[devops]: https://aws.amazon.com/devops/what-is-devops/
[git]: https://git-scm.com/
