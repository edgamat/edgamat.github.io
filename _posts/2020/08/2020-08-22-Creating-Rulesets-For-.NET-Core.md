---
layout: post
title: 'Creating Rulesets for .NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I found a useful Roslyn Analyzer for C# projects that added compiler warnings for some common coding issues. I didn't agree with all the default settings so I wanted to disable a few. Here is how I did it using a Custom Ruleset.

<!--more-->

The set of projects I currently work on all use C# .NET Core. We use Visual Studio and Resharper for all development. We have mostly informal coding standards, and overall, the code base follows them pretty consistently.

However, there is always room for improvement. Recently some new developers have joined the team and we ae re-assessing some of the previous coding conventions. One such convention is the use of braces around single line code blocks. The following are some acceptable statements using our current coding standards:

```csharp
if (request.IsValid == false)
    return BadRequest();

if (request.IsValid == false) return BadRequest();
```

The new developers were asking if this practice could be avoided and require all single line blocks to be enclosed in braces:

```csharp
if (request.IsValid == false)
{
    return BadRequest();
}
```

My first thought was to find a Roslyn Analyser that could identify these for us. And fortunately, there is one already available called [VSDiagnostics][VSDiagnostics].

To install this analyzer, use the Nuget package:

```powershell
dotnet add package VSDiagnostics
```

Once included in the project, any instances of single statement blocks without braces will result in the following compiler warning:

`warning VSD0023: An if statement should be written with braces`

But what if you wanted it to create an error, rather than a warning? this is where Rulesets come into play.

## Custom Ruleset Files

To add a Ruleset, create the following file (named `CustomRuleSet.ruleset`) in your project folder:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RuleSet Name="Custom Rule Set" Description="Custom Rules for My Project" ToolsVersion="15.0">
</RuleSet>
```

Then add a reference to this file in the CSPROJ file:

```xml
<PropertyGroup>
  <CodeAnalysisRuleSet>.\CustomRuleSet.ruleset</CodeAnalysisRuleSet>
</PropertyGroup>
```

Now we can start customizing the way the rules get interpreted by the compiler. If I wanted to change how the `VSD0023` rule is reported, add this to the ruleset file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RuleSet Name="Custom Rule Set" Description="Custom Rules for My Project" ToolsVersion="15.0">
  <Rules AnalyzerId="VSDiagnostics" RuleNamespace="VSDiagnostics">
    <Rule Id="VSD0023" Action="Error" />
  </Rules>
</RuleSet>
```

Now when the build is done, the result is a failed build with the following message:

`error VSD0023: An if statement should be written with braces`

Here are the possible severity levels:

Action (Severity) | Description
Warning | Generates a warning in the Error List and also at build time.
Error | Generates an error in the Error List and also at build time.
Info | Generates a message in the Error List.
Hidden | The violation is not visible to the user. The IDE is notified of the violation, however.
None | The rule is suppressed. The behavior is the same as if the rule was removed from the rule set.

You can then decide which rules you want to include or ignore.

For each analyzer, you will need to figure out the AnalyzerId and RuleNamespace. That is usually not difficult. Just look at the Nuget package or if available its source code on GitHub.

## Sharing Rulesets

If you have a multi-project solution, you may have some rules you want to share across all projects. You can do that using an include mechanism:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RuleSet Name="Rules for ClassLibrary" Description="Code analysis rules for ClassLibrary.csproj." ToolsVersion="15.0">
    <Include Path="Common.ruleset" Action="Default" />
    <Rules AnalyzerId="VSDiagnostics" RuleNamespace="VSDiagnostics">
        <Rule Id="VSD0023" Action="Error" />
    </Rules>
</RuleSet>
```

## Ruleset versus EditorConfig

It is now possible to use EditorConfig to perform the same function as rulesets:

```ini
# Code files
[*.{cs,vb}]

dotnet_diagnostic.VSD0023.severity = error
```

My experience so far is that using the EditorConfig approach is inconsistent. But it definitely is worth investigating further.

 [VSDiagnostics]: https://github.com/Vannevelj/VSDiagnostics
