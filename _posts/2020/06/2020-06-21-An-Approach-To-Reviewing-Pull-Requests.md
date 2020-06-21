---
layout: post
title: 'An Approach to Reviewing Pull Requests'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

As part of my job, I am often asked to review pull requests from other developers. Here are some ideas on my approach and things I have learned along the way. 

<!--more-->

## What is a Pull Request?

A [Pull Request][pr-def] is an opportunity for teams to collaborate on proposed changes to a project's source code. It allows the team members to discuss the changes, provide feedback to the author, and ultimately accept or reject the changes.

When a team uses Git, the majority of Git hosting solutions (GitLab, Bitbucket, GitHub, Azure DevOps) support flavors of the Pull Request (PR) process. A developer creates a branch in Git based on the branch they wish to change. For example, a team using [Gitflow][gitflow] might create feature branch based on the `dev` branch. Then, they create a PR which they can use to describe their changes and invite other team members to review the work. If the changes are accepted then the PR is 'completed' which merges the changes into the target branch.

While this process can vary from tool to tool, or team to team, I'd like to discuss how I conduct the review portion of this process. It doesn't rely much on the tools used or the host's flavor of Pull Request ceremony. It's mostly about how to be effective in these reviews and improve your team's ability to communicate.

## My Context

I currently work on a small team (4-6 members) with a mix of roles; analysts, developers, and testers. We follow a [trunk-based][trunk-based] development process. That means:

- We create changes based on one main branch of the code, called the 'trunk',
- The trunk branch is always release ready,
- We never break the build (which we automatically run on each commit to the trunk using a Ci/CD process), and
- All automated tests (also part of the Ci/CD process) must pass.

When the build breaks, everyone on the team stops what they are doing and assists getting the build fixed. When any automated tests fail, the same thing is expected. As a team, we coordinate efforts to ensure the tests get fixed before moving onto any other new work.

## My Approach

When a team member asks me to review a PR for them, the first thing to do is to read the PR. I need to understand the context of the changes. Is it related to a task for a User Story? Is it a bug-fix? Is it a technical change, like upgrading a Nuget package? Is it a configuration change for the testing or production environment? Knowing the context of the code changes is always the first step.

I encourage team members to always provide as much context as possible when they create a PR:
- A meaningful title/summary
- Indicate the user story the changes belong to (where applicable)
- Provide a description of the scope for the PR (is it the entire feature or a portion of a feature being developed)
- Provide any assumptions about the work or questions that the reviewers may need to consider about the code changes.

That way the review process can be more efficient and won't waste time trying to figure out what the changes are for.


### Download the Code, Must Compile, Must Pass all Unit Tests

Our trunk-based approach means that we need to run the build for the proposed changes, to prove that they don't break anything before being pushed into the trunk branch.

I download the branch to my local environment, build it, and ensure it compiles and all the automated unit tests pass. If something goes wrong at this stage of the review, I'll send a direct message to the author of the PR and work with them to resolve the issue(s). No point in continuing the review if it is clearly broken.

**NOTE** At this time, the project I am working on has an automated version of this running in our CI/CD pipeline. When a branch is pushed to the hosted repository, the system builds the branch and runs all the automated unit tests. If it fails, it broadcasts a message to the team's shared message channel so everyone can see there is an issue. It helps keeps us from starting a review too early if there is a known issue. 

### Must Run on my Local Environment

Next, I'll attempt to run the code in my local environment. It is a [smoke test][smoke-test] for the most part, but I want to verify that no unexpected side-effects have been introduced. Often I'll discover something that could only be found by actually running the code. This means the author of the changes either didn't attempt to run the code before submitting the PR, or has a different setup for their local environment. In either case, it is a good place to start collaborating with the author to resolve the issue and get the code fixed.

### Adhere to Agreed-Upon Standards

All effective teams agree to follow a set of standards. Most of these are formal, based on the best practices for the tools and programming languages being used. However, some are informal and might vary slightly from team member to team member. I usually will review the code looking for any inconsistencies to these standards and _ask the author_ to bring the code changes into compliance with the standards we all agreed to follow. 

Most of these violations are caught by the tools we use to develop with. However, some violations are not as easy to catch in an automated fashion, and can also vary a lot based on the programming language. For example, C# coding standards are easier to enforce with automated tools compared to SQL coding standards.

### Must Make Business Sense

Next, I look at the actual changes in behavior the PR makes. If it is fixing a bug, I make sure the bug is indeed fixed. If it is a change supporting a new feature, I make sure the changes match what's defined in the feature's User Story. If I find an issue, I _ask the author_ if they agree with my assessment and work with them to get the PR to match the expected behavior. Sometimes this boils down to a different interpretation of the expected behavior. So we may need to collaborate with other team members to get the situation resolved.

### Must Have Supporting Unit Tests (Where Applicable)

Most code changes can be supported by unit tests. In such cases, I look to see if the author has included them in the PR. Sometimes, they will, but not always. Sometimes, they will indicate in the PR description that the unit tests will follow in a future PR. While not an ideal way of doing things, it can be justified in some cases. For example, perhaps a PR is introducing an interface for a new service being written. Such a PR might be necessary in order for multiple team members to code against the interface. In this case, I would expect the future PRs that implement the interface to provide the supporting unit tests.

In some cases, a PR might only include configuration/setting changes. These wouldn't necessarily have supporting unit tests. So it really depends on the context.

That being said, in the vast majority of cases, I would expect to see unit tests for code changes. If they don't exist, I _ask the author_ if unit tests could be added to the PR.

I'll also inspect the unit tests to make sure they verify the expected changes in behavior. Sometimes the tests might not provide value because they don't verify the correct behavior. It is important that the tests are given the same level of review as the code. Otherwise, you cannot depend on them providing the feedback you need when they are passing/failing.

### Approve or Reject

The final step is either to approve or reject the PR. It is rare that I reject one. In fact I can't remember ever doing so. That represents a clear disagreement which probably is a communication issue more than a coding one.

When I approve a PR I sometimes want to provide some suggestions to the author. Fortunately most tools support this process and it makes it easy to do so.

## Providing Feedback

Providing feedback to a Pull Request is a responsibility I don't take lightly. When I started reviewing Pull Requests, I would look at the code changes and based on that alone I would provide feedback. But I found out pretty quickly that I needed to do more if I wanted the PR process to help my team produce better code.

In my opinion, the most important aspect of the PR process is how to give feedback. The PR is an expression of a co-worker's hard work. It represents something they undoubtedly feel is worthwhile reviewing. So the review process must treat this work with respect, and more importantly, treat the author with respect as well.

### Every PR is a Gift

Notice above when describing my review process I emphasized the phrase _ask the author_. I place importance on this point because every PR is a gift. It represents an opportunity to learn, to provide value, and to help solve problems for the project I am working on. What a great gift that is to receive. When I review the work someone presents, I don't want to tear it down, I want to rise it up and make it better. Asking the author to consider my opinion is much better than telling them what I want them to do. I may have missed a detail or may not have considered something that they know about the new feature. So I try to remain humble and ask the author for help in improving the code. 

### Be Kind

Having a good working relationship with your co-workers is very important for a team to be effective. And how you communicate can greatly affect that relationship. When giving feedback to the author of a PR, it is usually in written form. When receiving written feedback, it is difficult to interpret the tone of the feedback. You could easily be hurt by the feedback or mis-interpret the comment as criticism against you personally.

When giving feedback, I keep that in mind. Using phrases like these help communicate tone and intent in a constructive manner
- "I'm afraid that I don't agree with...", 
- "I wonder if this was changed to...",
- "Could you please change this to...",
- "Could you please help me understand this change..."

These examples are much better than using:
- "Change this to..."
- "This is wrong"
- "This makes no sense"

### Be Clear

I like to try to give as clear feedback as possible. If a feature doesn't work when I test it, provide the author the exact scenario that exposed the issue. Or if you are asking them to change something, give them a reason for the request:

POOR: "Remove this line"  
BETTER: "In our meeting I thought we agreed this was not needed? Could you please double check? If that is the case, could you please remove this from the PR?"

Is my 'better' response too long? Perhaps. But I'd rather lean to that approach more than the 'poor' approach of providing no context at all.

### Be Fair

It can be easy to let feelings creep into the review process. For example, you may feel frustrated that the same issues are showing up time and time again in pull requests. Someone isn't following the agreed-upon standards so you are always asking them to fix their code changes in the PR process. Or you can never get the code to behave correctly the first time you review the code and it requires a lot of back and forth with the author to get it actually running.

These are important concerns and indeed must be addressed for things to improve. However, the feedback in the PR should not be the place for such exchanges. You must keep the comments in the PR as neutral as possible. It must be a 'safe' place for co-workers to collaborate. If people are afraid of the feedback they might receive, it makes the PR impossible.

In fact, I sometimes see mistakes made by senior-level developers that I would expect them not to make. But I make sure my feedback to them is the same as it would be to a junior developer. Give feedback on the code, not the author. Any concerns you may have about the mistakes being made must be done elsewhere, one-on-one. The feedback in the PR is public, so everyone will see right away if any favoritism is being shown. So be fair.

### Be Punctual

Last, but not least, be punctual with your feedback. If someone submits a PR and is looking for your feedback, they probably want it sooner rather than later. If you can't get to it in a reasonable time, reach out to them directly and let them know. Maybe it's a critical change that will hold up other work from proceeding. But it might also be something that could wait a day before having it looked at. 

I typically will see PRs submitted later in the day. Not surprising as folks typically work towards getting tasks finished before they complete work for the day. So I usually will dive into reviewing the PRs first thing in the morning. I'm usually at work before anyone else so that arrangement works well. But everyone's schedule is different so you'll need to find a schedule for reviewing pull requests that makes most sense for you and your team. Just make sure everyone has the same expectations for the timeliness of the PR reviews being done.

## Summary

Reviewing pull requests is an important part of any process that relies on them to have work integrated into the main code-base of a project. It is a great way to communicate with team members and to share ideas. I have found a process that works well for me. I make sure the code runs and behaves correctly before I approve or accept the changes. I take care to give feedback in a constructive manner. I try to be kind with my comments, and most of all be fair so we can all benefit from the process.


[pr-def]: https://help.github.com/en/github/getting-started-with-github/github-glossary#pull-request
[gitflow]: https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow
[trunk-based]: https://trunkbaseddevelopment.com/
[smoke-test]: https://en.wikipedia.org/wiki/Smoke_testing_(software)
