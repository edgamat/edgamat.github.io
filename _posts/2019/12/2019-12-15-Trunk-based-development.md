---
layout: post
title: 'Using trunk-based development'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

For the past year I have worked on a project that uses trunk-based development. I thought I'd write down
some of my observations and ideas about my experiences.

<!--more-->

## What is Trunk-Based Development

According to the best source material for this approach ([trunkbaseddevelopment.com][ws]):

> A source-control branching model, where developers collaborate on code in a single branch called ‘trunk’ \*, resist any pressure to create other long-lived development branches by employing documented techniques. They therefore avoid merge hell, do not break the build, and live happily ever after.
>
> \* master, in Git nomenclature

For me, this is an accurate definition, to a point. I'm not sure we all lived happily ever after. We had our fair share of merge hell and break the build pretty regularly. However, we do all share our code changes using a single master branch and don't have any other long-lived development branches. So I'd say we are using this branching model but sill learning how to do it better.

**NOTE**: While this approach to development could apply in a number of scenarios and using various version control systems, in my case, we are using git, hosted on Azure DevOps \*, and use the built-in Pull Request support to
manage how changes are merged into the master branch.

\* DevOps, or Visual Studio Online, or Team Foundation Services, or Visual Studio Team System, or whatever it is called when you are reading this...

## Creating Branches

When I have a new task they are working on, I start off in the master branch and ensure I have all the latest code:

```powershell
git checkout master

git pull
```

I then create a private branch for my changes to live:

```powershell
git checkout -b {branch-name}
```

The naming convention my team agreed to follow for branch names is:

`{branch-name} = users/{userid}/{story-id}-{description}`

For example if I am creating a branch for story 123 which is adding a new validation rule to the system, the branch might be:

```powershell
git checkout -b users/matte/123-validation-rule
```

Now that I have a branch to work in, I start my changes to the code, making sure I update the version of the code (we store the version information in a script file that the CI/CD system picks up and uses to build the system for deployment). I also ensure I write unit tests that support the changes, either by following test driven development practices or by writing the tests immediate after the code is written.

Once the first set the changes compile and all tests are passing, I'll push the branch to the server, regardless if I have more changes to make or not, regardless if I am ready to create a Pull Request. I find that pushing the branch to the server is a great communication tool. It helps the other developers know what I am working on and how far along I am with completing the task.

```powershell
git push --set-upstream origin users/matte/123-validation-rule
```

**NOTE** - if you are using git on Windows, I strongly recommend using [posh-git][posh], a PowerShell module that integrates git with PowerShell. It integrates git status info into the PowerShell prompt and gives you tab-complete support for branch names. Very, very useful.

Once I have all changes made and all tests passing and pushed to DevOps, I'm ready to create a Pull Request. Right before I do that, I'll pull down into my local branch any changes made to the master branch since I created my local branch:

```powershell
git checkout master

git pull

git checkout users/matte/123-validation-rule

git pull origin master
```

And no, I don't use `git rebase`. The trail of the commits in the local branch get squashed during the Pull Request process so there is no point in using rebase. And I always checkout master, pull the changes down first. I tried pulling down all changes while still in the local branch and I found it to be unreliable. It didn't always set my local master branch correctly with the latest set of changes.

## Integrating Changes

To integrate my changes into the master branch, I create a Pull Request (PR) that includes a description of _what_ the changes are for and, when necessary, _why_ I have taken the approach I have to implement the change. The team decided to have optional reviewers on each PR, using a reasonable process, as not to slow down development. If the team lead is available (and he almost always is), assign it to him. Also include any other team member that you may feel has a stake in the changes and might be able to provide meaningful feedback.

The PR process expects each reviewer to confirm the following:

- the changes include an update to the version (major, minor, patch/revision). We follow [Semantic Versioning][semver] practices for our versioning:

  - Major: Breaking changes
  - Minor: New features, but backwards compatible
  - Patch: Backwards compatible bug fixes only

- the changes compile and all unit tests pass
- all integration tests pass (if present)
- the changes don't introduce a security flaw
- the changes fulfill the needs of the story/task
- the changes follow the coding practices the team agreed to use

### PR Size Influences Speed

To keep things moving quickly, we keep each PR small and cohesive. It may be that you have a lot of changes to make to complete a story, but all team members try to limit the scope of each PR to a single change. This improves the speed of the work getting done in a few ways:

- Each PR is small so the PR review process takes less time
- The time the local branch is active is shorter meaning fewer changes to the master branch would have occurred
- The rest of the team can see the changes I am working on earlier and integrate the changes into their work sooner

## My Experiences

This approach was difficult to get used to at first. I was used to following the [git-flow][git-flow] branching model. Not having a `develop` branch seemed wrong. Not having feature branches seemed to be dangerous. I admit I was not looking forward to this new approach.

However, what I soon discovered was the change in branching model required a fundamental change of thinking of my work. Using the new approach, I was going to create a PR for each set of changes. And I'd probably end up doing that more than once a day. And it was going right into the master branch, which was being automatically deployed to a shared integration environment and eventually the testing and production environments.

It forced me to be more sure of the changes I was making. I wrote more unit tests and used test-driven development more and more. I wanted to ensure the changes wouldn't break something in production so I relied more and more on the automated tests (unit and integration) to detect regressions.

It also meant I kept each PR more focused. If I need to improve the code via a refactoring, I would typically create a task for that purpose, and create a separate PR for just the refactoring. That kept each PR focused on a single idea of change to the code. This made most PRs just a handful of files. Not always, but mostly. And that made the PR review process easier on the reviewers, which meant they could get them reviewed more quickly.

## Feature Flags

A few times over the past year, we've had code changes we didn't want to be available yet in Production. We used feature toggles (boolean flags controlled via the environment settings) to enable or disable the features in each environment. It didn't occur often enough for us to use anything more sophisticated. And it worked well. When the feature was fully integrated and the flag was not longer necessary, we added a task to the work stream to have them removed.

## Continuous Integration

One aspect of our environment that supports trunk-based development is an automated Continuous Integration (CI) pipeline. Each change to the master branch is automatically built, tested and deployed to a shared integration environment. So throughout the day as we complete Pull Requests, we get a lot of feedback from the CI system that let's us know if we've introduced a problem, or haven't followed our process.

## Summary

I have enjoyed using the trunk-based development model. As a team, we are still learning how to do it better. Personally, I have a lot to keep improving on and have a lot still to learn. However, this approach to development has worked very well in the context my team and I are working in. I would strongly advocate trunk-based development in the future when circumstances are a good fit.

[ws]: https://trunkbaseddevelopment.com/
[posh]: https://github.com/dahlbyk/posh-git
[semver]: https://semver.org/spec/v1.0.0.html
[git-flow]: https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow
