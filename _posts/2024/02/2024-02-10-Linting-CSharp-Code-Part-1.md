---
layout: post
title: 'Linting C# Code - Part 1'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I spent a year and a half working in TypeScript. I learned a lot. I also enjoyed using linting code analyzers. Can I do the same in C#?

<!--more-->

## What is Linting?

Linting is the process of using a static code analysis tool (a 'linter') to identify code style issues, formatting inconsistencies, bugs, etc. in your code.

I first used `tslint` with Angular, and then `eslint` with NodeJS and TypeScript. 

After you finish making your code changes, you run the linter and it will scan all your files for potential rule violations to address. These rules are configurable, and you can decide which ones to ignore or which ones to enforce. The linters can also fix some the issues too, making it simple to enforce certain coding standards.

In the .NET world, my first experience with this type of tool was StyleCop. It was a Visual Studio extension that would enforce coding standards in C# code. My experiences with it were mixed. At first I liked it a lot because it forced everyone to follow a particular standard. But I eventually used it less and less on projects. I found it interfered too much while I was coding. It would be better if I could use it after I had finished coding, like `eslint`. And honestly, there were more rules that I disabled than I enabled... it didn't seem like a good thing any longer.

I am revisiting this process many years later in the .NET world of 2024. I want to see if I can achieve the same TypeScript linting experience with C#.

## The Linting Process

Each team needs to decide on their own process for enforcing coding standards. For code linting, there are usually two choices. First, you can apply the linting rules as part of the build process. If there is a rule violation, then the build fails. Or the alternative is to wait and apply the linting rules as a separate step outside of the build. Developers can apply formatting fixes after they have written the code.

The first option leaves you little choice. You have to fix the rule violations as you go. This is the way StyleCop works. I find this approach to be too intrusive. It disrupts your flow when coding. Don't misunderstand, I think that applying the rules is very important. But if you are constantly worrying about formatting issues while you are coding, it can be very distracting.

I prefer the second option. This is how eslint works with TypeScript. Once you are finished coding, you run the linter. It can fix the errors for you (well, it fixes what it can. It doesn't always fix every rule violation). Or you can run it and let it show you what the violations are, and you can fix them manually. 

The important part is in the automated CI/CD pipeline. When you are committing new changes to the main branch, no rule violations must be present in the code changes. Your CI/CD pipeline must include a step that confirms that all the linting rules pass. 

If you forget to fix the formatting, the pipeline fails. This lets you know that you need to fix the formatting, similar to how the tests are treated in the pipeline. Failing test = failing pipeline.

Most CI/CD systems (GitLab, Azure DevOps, GitHub Actions) include the ability to prevent a merge of code into the main branch unless all steps are successful. Including a linting step will ensure the linter verifies the code is free of any rule violations before merging the changes to the main branch.

## Let's See What .NET Can Do

The first step in establishing a linting process with .NET is to determine what the rules should be. For .NET 6.0 and later, the way to do this is to store them in an `.editorconfig` file. The .NET SDK provides you with a template:

```bash
dotnet new editorconfig
```

This command will generate a new `.editorconfig` file based on the default rules found in Visual Studio. Once you have this file, you can customize it to suit your team's coding conventions.

To ensure any formatting rule violations are fixed, you need to add the following to the `.editorconfig`:

```
# IDE0055: Fix formatting
dotnet_diagnostic.IDE0055.severity = error
```

Next you need to run the linting tool. In the case of .NET, it is called `dotnet format` and can be run like this:

```bash
# List rule violations (but don't try to fix them):
dotnet format -v detailed --verify-no-changes

# List (and fix) rule violations:
dotnet format -v detailed
```

A description of the tool and its options are found here:  
[https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-format](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-format)

If you run the `dotnet build` command, these linting rules are not enforced. To do that, you need to add a couple parameters:

```bash
dotnet build /p:EnforceCodeStyleInBuild=true /p:GenerateDocumentationFile=true
```

With these additional parameters, any code style violations will raise compile-time warnings/errors.

When a developer is coding locally, they can use the `dotnet build` command (or build the code in Visual Studio, VS Code, Rider, etc.) and the code style warnings are not enforced. When they are ready to create a pull request, they can run the `dotnet format` command and have the rules applied to their code. However, in the CI/CD pipeline, we can include the additional EnforceCodeStyleInBuild/GenerateDocumentationFile properties with the build command and ensure no formatting issues remain unresolved prior to merging the changes. 

## Fix Formatting on Save

Microsoft Visual Studio 2022, VS Code and JetBrains Rider all provide the means to fix any formatting violations on save. For those who prefer to work this way, it can be a great middle ground. You don't have to worry about rule violations because the IDE fixes them for you. And you don't have to get taken out of your flow to deal with them. You simple save the file and the fixes are made automatically.

## Summary

Linting your C# code is indeed possible, and flexible in how you want to enforce the rules. It is a great way to promote consistency in the codebase. The automatic enforcement of the rules means one less thing that needs to be part of any code reviews, saving the team time.
