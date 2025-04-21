---
layout: post
title: 'Scripting .NET CLI CI/CD Commands'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

You submit your code for review and the CI/CD pipeline for your branch fails. I wish I could have
known about it before I submitted the PR!

<!--more-->

Our Continuous Integration/Continuous Delivery (CI/CD) pipeline using the .NET CLI to build and
test the code. It also does some code quality checks to look for formatting violations and potential
security vulnerabilities. We often find that builds will fail due to some of code quality checks
because these commands are not developers have at their disposal when they are change the code on
their local PC.

Our goal is to change that. We want developers to have a means to run the same commands on their
code branch prior to triggering a CI/CD pipeline to run. I am going to refer to this as the 'Check'
step: the final thing a developer should do before triggering a pipeline is _check_ that their
code meet all the same expectations that the pipeline has. They can make any corrections
to their code before triggering the pipeline. This saves time not only for the developer but also
for reviewers of the code.

### The Build / Test Pipeline

Here are the steps I want to run, and their dependencies.

- First I want to build the source code using `dotnet`, the .NET CLI.
- If successful I want to run all the unit tests.
- Next, I want to check for vulnerable package references.
- Lastly, I want to check for any formatting errors.

Here's what it would be in a dependency graph:

```text
                 [Build]
                    |
        +-----------+------------------+
        |           |                  |
     [Test]   [Security Scan]   [Formatting Check]
```

Very straightforward. The 'Check' task should depend on the 3 non-build steps.

### Attempt #1 - A PowerShell Script

I created a PowerShell script that is a poor-man's dependency graph:

```powershell
# check.ps1
function RunBuild {
    dotnet build
}

function RunTest {
    dotnet test --no-restore
}

function RunFormatting {
    dotnet format -v detailed --verify-no-changes --no-restore
}

function RunSecurity {
    dotnet list package --vulnerable > vulnerable.log
    $vulnerable = Select-String -Path "vulnerable.log" -Pattern "has the following vulnerable packages" -Quiet
    if ($vulnerable) {
        Write-Output "Security vulnerabilities found in the command output."
        cat .\vulnerable.log
    }
}

RunBuild
RunTest
RunFormatting
RunSecurity
```

It works... but it has its shortcomings. First, I can't run the tests, formatting and security steps
separately. I can only run the scripts together. That's not too difficult to fix.

### Attempt #2 - Another PowerShell Script

IN this script, we will pass in a command parameter which allows us to select which checks we want
to perform:

```powershell
param (
    [string]$Command = "check"
)

function RunBuild {...} # collapsed

function RunTest {...} # collapsed

function RunFormatting {...} # collapsed

function RunSecurity {...} # collapsed

switch ($Command) {
    "build" {
        RunBuild
    }
    "test" {
        RunBuild
        RunTest
    }
    "format" {
        RunBuild
        RunFormatting
    }
    "security" {
        RunBuild
        RunSecurity
    }
    "check" {
        RunTest
        RunFormatting
        RunSecurity
    }
    default {
        Write-Host "Command not recognized"
    }
}
```

Now I can run each of the checks separately as well as combined. But there is the problem now of running
the build multiple times when running them all combined. What I need is a way to run the build only
once when running them combined.

### Attempt #3 - GNU Make

Make is a command line tool that executes programs based on a dependency graph defined in a 'makefile'.

You can download make here: <https://gnuwin32.sourceforge.net/packages/make.htm>

The makefile defines the targets and their dependencies. By default it will run the first target.

Here is what mine looks like:

```makefile
# Define targets and their dependencies
.PHONY: check
check: build test security format

# Step 1: Build the .NET project
.PHONY: build
build:
  dotnet build

# Step 2a: Run unit tests
.PHONY: test
test: build
  dotnet test --no-build

# Step 2b: Check for vulnerable packages
.PHONY: security
security: build
  pwsh -NoProfile -ExecutionPolicy Bypass -File ./security-check.ps1

# Step 2c: Check formatting issues
.PHONY: format
format: build
  dotnet format --verify-no-changes --no-restore
```

`./security-check.ps1` is the script that for vulnerable packages. It was not easy to run the script
inline within the makefile so I moved it to a script file.

### Attempt #4 - Bullseye/SimpleExec

I recently watched a video on YouTube about using these types of tools. It presented a C# package
that could be used instead of `make`. The benefit is that defining the targets and their dependencies
is done in C#. One language everywhere!

<https://github.com/adamralph/bullseye/tree/main?tab=readme-ov-file>

<https://github.com/adamralph/minver?tab=readme-ov-file>

Following the instructions:

We create a new project in the solution called "Targets":

```powershell
dotnet new console -n Targets
cd Targets
```

Then add a reference to the 2 NuGet packages:

```powershell
dotnet add package Bullseye
dotnet add package SimpleExec
```

The `Program.cs` defines the targets and their dependencies.

```csharp
using static Bullseye.Targets;
using static SimpleExec.Command;

Target("default", dependsOn: ["test", "security", "format"]);

Target("build", 
    () => RunAsync("dotnet", "build"));

Target("test", dependsOn: ["build"], 
    () => RunAsync("dotnet", "test --no-build"));

Target("security", dependsOn: ["build"], 
    () => RunAsync("pwsh", "-NoProfile -ExecutionPolicy Bypass -File ./security-check.ps1"));

Target("format", dependsOn: ["build"], 
    () => RunAsync("dotnet", "format --verify-no-changes --no-restore"));

await RunTargetsAndExitAsync(args, ex => ex is SimpleExec.ExitCodeException);
```

To run it, use the following command:

```powershell
dotnet run --project .\Targets\Targets.csproj
```

To run a single target:

```powershell
dotnet run --project .\Targets\Targets.csproj -- test
```

### Attempt #5 - Psake

Psake (<https://psake.dev/>) is another build automation tool, written in PowerShell. It can be installed
as a module:

```powershell
Install-Module psake
```

And here is the equivalent script using psake:

```powershell
# psakefile.ps1

Task Default -Depends Build, Test, Security, Format

Task Format -Depends Build {
    & dotnet format -v detailed --verify-no-changes --no-restore
}

Task Security -Depends Build {
    & dotnet list package --vulnerable --include-transitive > vulnerable.log
    ./security-check.ps1
}

Task Test -Depends Build {
    & dotnet test --no-build
}

Task Build {
    & dotnet build
}
```

### Which Option is Best?

As always, it depends. If you are only going to use these 'check' files locally and don't need to
run these scripts as part of your CI/CD pipeline, then `psake` would be my choice. It is easy to
set up, easy to read and gives you the full power of PowerShell to script the steps.

However, for others, the choice might be driven on the tools used to execute the CI/CD pipeline.
You might prefer to use the same script in the CI/CD pipeline, in which case you would need to install
`psake` or `make`.

However, if the CI/CD script is running the tasks one after the other, then the original script
(Attempt #1) could be all you require.

### Summary

There are lots of choices here and you will need to find the one that best works for oyu and your context.

But the gaol is really what is important. We want to allow developers to find potential CI/CD issues
before attempting to run the pipeline. It is more efficient and makes for a better experience for
everyone.