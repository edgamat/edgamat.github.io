---
layout: post
title: 'Linting C# Code - Part 2'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In Part 1, I showed how to lint your C# code using the `dotnet format` command and storing the rules in a `.editorconfig` file. Let's see how we can modify the rules to suit our coding style.

<!--more-->

Let's start with a console application. I added a `Person.cs` file:

```csharp
namespace LintingCSharpApp;

public class Person
{
    public string firstName { get; set; } = ""; // <-- Naming rule violation

    public string LastName { get; set; } = "";
}
```

The `Program.cs` looks like this:

```csharp
using System; // <-- Unnecessary using directive

using LintingCSharpApp;

var person = new Person
{
    firstName = "John",
    LastName = "Doe"
}; 

if(person.firstName ==        "John")  // <-- Whitespace violations
{
    Console.WriteLine("Hello, John!");
}
else
{
    Console.WriteLine("Hello, stranger!");
} 
```

## Fix Formatting

The first set of rules we want to see violations for are formatting related. They are described here:

[https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0055](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0055)

This page describs rule IDE0055: "All formatting options have rule ID IDE0055 and title Fix formatting." In other words, regardless of what formatting option you violate, it will be reported as a violation of rule IDE0055.

To ensure any formatting rule violations are fixed, you need to add the following to the `.editorconfig`:

```
# IDE0055: Fix formatting
dotnet_diagnostic.IDE0055.severity = error
```

The following line from the above example would show 2 violations:

`if(person.firstName ==        "John")  // <-- Whitespace violations`

```
C:\LintingCSharpApp\Program.cs(11,3): error WHITESPACE: Fix whitespace formatting. Insert '\s'.
C:\LintingCSharpApp\Program.cs(11,24): error WHITESPACE: Fix whitespace formatting. Delete 13 characters.
```

You can fine-tune the formatting options to suit your needs by looking at the documentation for the options associated with rule IDE0055. For example, the `dotnet_sort_system_directives_first` option determines how the using statements should be sorted. You can find it in the `.edtiorconfig` and see that, by default, it is set to `true`. But if your team perfers that System namepace usings be included with all other namespaces when sorting, then you can set this to `false`.

Once you have the IDE0055 severity level set to `warning` or `error` you will see the following violations:

```
C:\LintingCSharpApp\Person.cs(3,14): warning CS1591: Missing XML comment for publicly visible type or member 'Person'
C:\LintingCSharpApp\Person.cs(5,19): warning CS1591: Missing XML comment for publicly visible type or member 'Person.firstName'
C:\LintingCSharpApp\Person.cs(7,19): warning CS1591: Missing XML comment for publicly visible type or member 'Person.LastName'
```

If you don't use these comments in your code, you can diable this rule by adding the following to your `.editorconfig` file:

```
# CS1591: Missing XML comment for publicly visible type or member
dotnet_diagnostic.CS1591.severity = none
```

## Enforce Code Styles

Code style rule IDE0005 deals with unused using directives. They can be removed without changing how the code runs. If you want to require they be removed, you can set the rule severity in the `.editorconfig` file:

```
# IDE0005: Using directive is unnecessary.
dotnet_diagnostic.IDE0005.severity = error
```

The following line from the above example would show 1 violation:

`using System;`

```
C:\LintingCSharpApp\Program.cs(1,1): error IDE0005: Using directive is unnecessary.
```

A description of the rule is found here:

[https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0005?pivots=lang-csharp-vb](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0005?pivots=lang-csharp-vb)

## Enforce Naming Styles

All the above rules have corresponding code fixes in their analyzers. If you run `dotnet format` the formatting is automatically fixed for you. Let's look at a rule that doesn't include a code fix. It is rule IDE1006, Naming Styles. In the above example, the `firstName` property doesn't adhere to the 'Public member capitalization' naming convention. 

You can treat such violations as errors with the following addiion to the `.editorconfig`:

```
# IDE1006: Naming Styles
dotnet_diagnostic.IDE1006.severity = error
```

However, when you run `dotnet format` you get the following message:

```
dotnet format
C:\LintingCSharpApp\Person.cs(5,19): error IDE1006: Naming rule violation: These words must begin with upper case characters: firstName
  Running 1 analyzers on LintingCSharpApp.
Unable to fix IDE1006. Code fix NamingStyleCodeFixProvider doesn't support Fix All in Solution.
```

In other words, you have to correct this one yourself. This is, I assume, because it is not possible for the static code anaylzer to understand the context well enough to make the change on your behalf. Thankfully, VS Code, Visual Studio, and Rider all have built-in refactorings for the types of renaming changes, making it straightforward to fix.

More details can be found here, including examples on how to adjust the naming conventions or add new ones:

[https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/naming-rules#rule-id-ide1006-naming-rule-violation](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/naming-rules#rule-id-ide1006-naming-rule-violation)

## Summary

I've shown a few ways to customize the rules in the `.editorconfig` file to suit your coding style, or your team's coding style. I reccommend trying them out. If can be a gret way to promote consistency and reduce the issues being discussed during code reviews.
