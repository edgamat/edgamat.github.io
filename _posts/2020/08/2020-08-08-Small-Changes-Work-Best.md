---
layout: post
title: 'Small Pull Requests Work Best'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

One of the reasons that Agile software development approaches work so well is the way the work is performed in small iterations. Breaking down complex tasks into smaller pieces means that each piece is much simpler to define (ie. requirements), implement, and test. The same goes for code changes submitted as a [Pull Request][pr-def]. It takes practice to work with small sets of changes, but the benefits are worth it.

<!--more-->

## The Context

### Our Process

I work with a team that follows a Scrum-based methodology to organize our work streams. We have a project stakeholder that creates and prioritizes user stories and a multi-discipline team who works the stories in two-week Sprints. 

### Pull requests

When a developer selects a task to work on, he/she will make changes to the code and then submit these changes for review using a Pull Request (PR). Each PR is reviewed by one or more people on the team. Once the review is complete and the changes are approved, they are committed to the main trunk of our software repository and published for further testing before ultimately being deployed into Production.

## Make Small Changes

Over time, we have learned that keeping the scope of each PR as small as possible has positive benefits.

In practice, this means that we aim to submit multiple Pull Requests for review each day. Each PR is meant to be a cohesive set of changes, not a series of unrelated changes.

### Keep Each Pull Request Focused on One Thing

For example, let's suppose the task being worked is adding a new HTTP endpoint to an HTTP API project. As you apply the changes, you notice that there are some inconsistencies in some existing property and method names. You want to improve the code and change these elements so they follow the agreed-upon conventions. The urge to include these changes in your PR with the new HTTP endpoint is understandable, but it is best to refrain from including them.

Including a series of refactoring changes in a PR can make it confusing to review. What changes are intended to be part of the new functionality versus the refactoring can be difficult to separate. The advice I would offer is to keep refactoring changes separate. Create a 'refactoring' PR before or after the 'real' PR you want to submit for your current task. That way the reviewers can focus on the purpose for each PR and give meaningful feedback.

### Create Multiple Pull Requests Per Story

It can be tempting to wait until all the work for a given Story is complete, fully covered by unit tests, before submitting a Pull Request. This is sometimes reasonable for smaller stories. But for most stories, it can represent a dozen or more files being changed or added. It is more efficient to submit Pull Requests as you work the story.

For example, let's say you've been working on a task and you've come up with a good solution to satisfy the requirements. You make all the code changes and it took you most of the day to get them completed and fully tested. You submit your PR as you leave work for the day. No one looks at the PR until the next morning. Someone notices that the approach that you've taken has missed a key part of the requirements outlined in the User Story. You end up having to go back through and rework your solution and because of all the new changes, you don't get the work done until the end of the day. It has now taken you two days to get an acceptable Pull Request.

It may have been more efficient to submit a simpler PR outlining your approach mid-way through the first day you were working on the story. It would allow you to receive the feedback earlier and possibly avoid having to rework your solution.

## What is the Power of Pull Requests?

In my opinion, the most valuable benefit of using an Agile process is the ability to receive feedback as early as possible so you can learn and understand more about the work being done. 

I think this is an important part of using Pull Requests that gets missed. Pull Requests shouldn't be a formal, drawn out, process of making sure everything is perfect. 

__A Pull Request is a Communication with Your team__

Pull Requests are a great way to let your team know what you are up to. It gives them a chance to help you get your work done more quickly, with fewer issues. It is a way to learn more about the task you are working on. Get feedback, incorporate it into your work, and repeat.

Some developers rarely include a description with the Pull Request. I feel this is no different than sending an email to a co-worker that includes a subject and an attached document. The body of the e-mail message is basically "Here!". No one would consider that 'effective communication' and it is rare for anyone to send such a message. This is not a learning mentality. 

I have heard some developers explain that the code should 'speak for itself'. I, as a reviewer, should be able to make sense of the changes without any additional context. In my experience, that is rarely possible. I want to understand _WHY_ the changes were made. The description of the Pull Request should include some explanation of why the Pull Request exists at all. The author of the Pull Request should respect the time of the reviewers and provide them a reasonable amount of context so a reviewer doesn't waste their time trying to understand things that are 'obvious' to the author.

## Humility versus Pride 

I have spent years trying to be a good software developer and continually improve. I like to help people and I like to learn. I am proud of the work that I perform. This pride has gotten me into trouble at times. It is something that I always struggle balancing with humility. But it is really about respecting the Way of the Universe:

__You Always Know Less than You Think You Do__

Keeping this in mind, remaining humble, is the best place to start learning and understanding.  When I submit a Pull Request to my co-workers, I expect to learn something. My pride might make me think that I am submitting the Pull Request to show the rest of the team how good a developer I am. But the reality is that the Pull Request is likely to contain errors or omissions. I am likely going to have changes to make. 

## Conclusion

I work with a great team. We all have each other's interest at heart and we look for ways to help each other. Each Pull Request is a chance to let them help me be better. What a great gift. I'd be stupid not to take advantage of such an opportunity. Regardless of the experience of each team member, they all have something to contribute. 

To receive this benefit, I work in small, related code changes. It gives me the feedback more quickly, and it makes the job of the reviewer easier. I call that a win-win.



[pr-def]: https://help.github.com/en/github/getting-started-with-github/github-glossary#pull-request
