---
layout: post
title: 'Linting C# Code - Part 3'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Let's see what one possible workflow looks like for linting your C# code.

<!--more-->

As I wrote in Part 1 of this series, linting is the process of using a static code analysis tool to identify code style issues, formatting inconsistencies, bugs, etc. in your code.

There are several ways to introduce linting to a .NET project. Every team will need to decide what works best for them. What I am showing here is something that I feel works best for me. My ideas on this process have changed from where they were in the past. and I'm sure they will change in the future. But for now, this is what I think makes the most sense.

## The Linting Process

I'll start a set of changes by creating a branch in `git`. In the IDE (Visual Studio, Rider, etc.) the linting rules are not enforced as errors. But some of them, if not all will appear as suggestions or warnings. I am free to format my code, use naming styles, etc. however I feel. But once it is time to integrate my changes with the main branch, that's when I have to ensure my code changes adhere to the agreed upon coding rules.

We have codified these rules in an `.editorconfig` file and checked it into `git`. That way everyone on the team uses the same set of rules (and so does the CI/CD pipeline). To prepare my branch to be merged into the main branch, I run the following:

```powershell
dotnet build
```

This does a build of all my code.

Next, I check for any linting rule violations using the following command:

```powershell
dotnet format -v detailed --verify-no-changes
```

If any linting rules are violated by my code changes, it will provide me with a list of what needs to be fixed. Most of the linting violations are accompanied by code fixes that can be applied automatically. You can try this command to auto-correct the violations:

```powershell
dotnet format -v detailed
```

You can then recheck your code for any remaining violations:

```powershell
dotnet format -v detailed --verify-no-changes
```

Once you have finshed fixing all the violations, you are ready to merge your changes into the main branch.

## Include Scripts for Linting Commands

Memorizing all these commands is not worth our time. It is much better to create scripts to run this for us. Here is a simple script that can be used on almost any .NET project (.NET 5 or later).

```powershell
# make.ps1

param (
    [string]$Command
)

function RunBuild {
    dotnet build
}

function RunRebuild {
    dotnet build --no-incremental
}

function RunTest {
    dotnet test --no-restore
}

function RunLint {
    dotnet format -v detailed --verify-no-changes --no-restore
}

function RunLintFix {
    dotnet format -v detailed --no-restore
}

switch ($Command) {
    "build" {
        RunBuild
    }
    "rebuild" {
        RunRebuild
    }
    "test" {
        RunTest
    }
    "lint" {
        RunLintFix
    }
    "lint-fix" {
        RunLint
    }
    "check" {
        RunTest
        RunLint
    }
    default {
        Write-Host "Command not recognized. Use 'build', 'rebuild', 'test', 'lint', 'lint-fix', or 'check'."
    }
}
```

To run your linting process, use these commands:

```powershell
# build your code
.\make.ps1 build

# check for linting violations
.\make.ps1 lint

# fix linting violations
.\make.ps1 lint-fix
```

## Summary

Linting your C# code is possible, with many ways of achieving the goal of consistent, well formatted code. Chose which rules you want to enforce, and codeify them in an `.editorconfig` file. Then use the built-in tools that come with .NET to help everyone on your team knows how to check for violations and how to fix them. 
