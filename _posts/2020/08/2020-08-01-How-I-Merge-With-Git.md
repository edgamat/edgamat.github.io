---
layout: post
title: 'How I Synchronize Changes With Git'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

As with most things, there are more than one way to accomplish a task. But some are better (simpler or easier or faster) than others. I'd like to share my approach to synchronizing my local branch with Git. It is not the 'best' way or the 'only' way, but it works well for me.

<!--more-->

## The Context

My typical scenario is when I have changes in a local branch that I have been working on. I want to incorporate changes made to the branch I intend to merge with. Let's assume the branch I am working on is `users/matte/my-new-feature` and it is going to be merged with the `main` branch, which is the main trunk of our development workflow. 

When I am ready to create a new Pull Request, I want to ensure that any changes that have been made to the `main` branch are pulled into my feature branch prior to creating/completing the Pull Request. 

Here is how I create a new feature branch. I **checkout** the branch I want to change, in this case the `main` branch. I then **pull** down all the latest changes from the remote repository and **create** my new feature branch.

```powershell
c:\git\my-project> git checkout main
c:\git\my-project> git pull
c:\git\my-project> git checkout -b users/matte/my-new-feature
```

## The Workflow

After I have worked on my local branch and committed changes to it locally, I'll want to start the Pull Request process. I'll want to make sure that I have all the latest changes to the `main` branch incorporated into my feature branch before the Pull Request is created. Let's walk through that process.

### Step 1: Prepare for the Merge

The first thing I want to prepare is the 'receiving' branch, in this case, that's the `main` branch. I need to ensure that my local repository has all the latest changes that have been committed to that branch. I do that by checking out that branch and pulling the latest changes.

```powershell
c:\git\my-project> git checkout main
c:\git\my-project> git pull
```

### Step 2: Pull the Changes

I need to merge in all the latest changes into my feature branch. I do this with the following commands:

```powershell
c:\git\my-project> git checkout users/matte/my-new-feature
c:\git\my-project> git pull origin main
```

The `git pull origin main` command fetches all changes from the main branch and then merges them into the feature branch.

### Step 3: Resolve Conflicts

If the merge cannot be performed automatically, then Git will indicate that a merge conflict has occurred. At this point we need to fix these conflicts manually. I won't go into the specifics, but there are many ways to do this. If you are using an IDE for Git, there is usually support for visually reviewing merge conflicts and resolving them.

For example, in my workflow with C# projects using .NET Core, the IDE I use is either JetBrains Rider or Visual Studio. Both of these include full support for resolving merge conflicts.

### Step 4: Verify the Changes

One all the conflicts are resolved, you commit the changes to your local branch with a commit message:

c:\git\my-project> git commit -m "merge changes from main branch"

### Step 5: Verify the Merge

Once the latest changes to the `main` branch have been merged into your feature branch, you must verify that all the changes still work as intended. I always perform a full recompile of all my code, rerun all unit tests, and run smoke tests. This makes sure that incorporating the latest changes from the `main` branch hasn't introduced a problem.

### Step 6: Push my Feature Branch

Once I have verified that I have a stable branch after the merge, I push my changes to the remote repository:

```powershell
c:\git\my-project> git push
```

### Step 7: Start my Pull Request

Depending on the host I am using for Git, I start the Pull Request (PR) process. And then the PR gets reviewed by my co-workers and we make sure the changes are acceptable to be merged into the `main` branch.

## Lessons Learned

### Merge Early, Merge Often

I try to keep the time I have a feature branch in progress as short as possible. I want to create a feature branch, make changes, create the PR and complete the PR as quickly as possible. Ideally, I want to do it all in one day. Even multiple times a day. But that is not always possible.

The longer I have a feature branch in my local repository, the more likely changes are going to occur to the `main` branch. I therefore make it a habit to keep my local `main` branch up to date. I checkout the `main` branch and pull the latest changes down anytime I see that changes have been made. Our development team has a messaging app that sends out messages to the entire team whenever changes are committed to the `main` branch. 

Keeping the `main` branch up to date is only half of the procedure. then I need to synchronize those changes with my feature branch. Doing this often reduces the chance of a merge conflict and reduces the difficulty of performing the merges. 

### Merge Changes Before Completing a Pull Request

The other lesson I have learned is to always perform one last merge before completing a Pull Request. While my code has been under review, often there have been changes made to the `main` branch. Just like when I create the PR, I merge all the latest changes into my feature branch prior to completing the PR.

Sometimes, incorporating the changes requires me to modify the behavior of my feature branch. If this occurs, I usually ask the reviewers to give the PR another check. This makes sure that any of these last-minute changes don't cause any issues.

## Summary

Managing the synchronizing of changes is very important. Following these steps, I have found a process that works well for me. I can repeat these steps without too much effort. It prevents me from taking too much time getting my changes merged into the main branch.




