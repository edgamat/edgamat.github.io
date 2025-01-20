---
layout: post
title: 'Creating a Custom .NET CLI Tool'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Installing tools on Windows can sometimes be challenging. The .NET CLI has a great way to distribute
tools to your team.

<!--more-->

Often it is useful to distribute command line tools to everyone on your .NET project. It can help
improve their productivity and promote consistency. It is a great experience for onboarding new
team members when you can point to them to a tool that encapsulates what normally is a series of
manual steps.

## .NET Tools

A .NET tool is a special NuGet package that contains a console application. You can install it as a
global tool which means it can be accessed from anywhere on your PC via the command line.

Global tools are installed in the following folder (on Windows or Linux):

```bash
~/.dotnet/tools
```

This folder is (by default) added to the %PATH% environment variable, which makes any programs in
this folder accessible from any shell session.

## Creating a Custom .NET Tool

For this example, let's create a custom tool that creates a random Guid and prints it to the
standard output.


A .NET tool is a console application, distributed using a NuGet package. We start by creating a new
Console app:

```bash
dotnet new console -n GuidGenerator
```

Then add the following to the CSPROJ file:

```xml
  <PropertyGroup>
    <PackAsTool>true</PackAsTool>
    <ToolCommandName>gg</ToolCommandName>
    <PackageOutputPath>./nupkg</PackageOutputPath>
    <Version>1.0.0-alpha1</Version>
  </PropertyGroup>
```

The `ToolCommandName` property is what determines the name of the tool when it is installed. In this
case, setting the value to `gg` means the tool will be installed as `gg.exe` on Windows (or `gg` on
Linux).

Now we update the `Program.cs` file to print the Guid:

```csharp
// Program.cs
Console.WriteLine(Guid.NewGuid().ToString());
```

We now pack the tool into a NuGet package:

```bash
dotnet pack
```

## Install a Custom .NET Tool

At this point, the NuGet package is stored in the `./nupkg` folder. To install the tool locally, you
can run this command:

```bash
dotnet tool install --global --add-source ./nupkg GuidGenerator --prerelease
```

This can be useful when testing the tool before deploying it to a NuGet feed.

To list the installed tools, use this command:

```bash
dotnet tool list --global
```

```text
Package Id                           Version           Commands
----------------------------------------------------------------
guidgenerator                        1.0.0-alpha1      gg
```

To run the tool, you can now call it from the command line:

```bash
C:\Tools> gg
cdd14ae2-cd01-4672-afe4-d85d58b7032f
```

To uninstall it, use this command:

```bash
dotnet tool uninstall --global GuidGenerator
```

## Parsing Command Line Arguments

The .NET CLI has a well designed interface where the functionality is presented as commands, which
themselves can have sub-commands. You can create a similar experience for your custom .NET tool. The
interface the .NET CLI uses is based on an abandoned library called `System.CommandLine`:

[https://learn.microsoft.com/en-us/dotnet/standard/commandline/](https://learn.microsoft.com/en-us/dotnet/standard/commandline/)

You can use the preview NuGet package or take a copy of the code and use it in your own project,
respecting the conditions in the LICENSE file in the source code repo:

[https://github.com/dotnet/command-line-api](https://github.com/dotnet/command-line-api)

```bash
dotnet add package System.CommandLine
```

Another option is the `Spectre.Console.Cli` library:

[https://spectreconsole.net/](https://spectreconsole.net/)

There are other libraries you can find, so try them and find one that suits your needs and context.

## Summary

The .NET CLI has a great feature to install your own custom tools using NuGet packages. Try it out!
