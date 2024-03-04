---
layout: post
title: 'Custom .NET Templates'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I create new projects all the time to test out ideas. I wish I could have a default solution template that was closer to reality than what the default templates offer. Let's build one.

<!--more-->

### .NET Templates

The .NET SDK provides numerous templates that can be used via the `dotnet` command line tool:

```powershell
# List installed templates
C:\templates> dotnet new list

# Create a new console project
C:\templates> dotnet new console
```

What I want is something more advanced, a combination of these templates to create a more complete solution:

- separate folders for source files and test files (`src` and `test`)
- separate "UI" and "Domain" projects
- initialize the same user secret id for the "UI" and "Domain" projects
- a test project for each source project
- a solution file
- an `.editorconfig` file
- a `.gitignore` file
- a `globaljson` file

I have been using a PowerShell script for this purpose. Here is a sample:

[https://gist.github.com/edgamat/29be20f87a0f94614597d6e93bc265f9](https://gist.github.com/edgamat/29be20f87a0f94614597d6e93bc265f9)

But what I would like to try is creating a custom template I can use via the `dotnet` command line tool.

### Create a Custom Template

There's a set of instructions for creating custom templates, so I'm going to follow them:

[https://learn.microsoft.com/en-us/dotnet/core/tools/custom-templates](https://learn.microsoft.com/en-us/dotnet/core/tools/custom-templates)

I used my PowerShell script to create a template solution:

```powershell
C:\templates> create-webapi.ps1 mytemplate
```

I added a `.template.config` folder and created a `template.json` file:

```json
{
  "$schema": "http://json.schemastore.org/template",
  "author": "Matt Edgar",
  "classifications": [ "Web API" ],
  "identity": "Edgamat.WebApiTemplate.CSharp",
  "name": "Web API Application",
  "shortName": "edgamat-webapi"
}
```

According to the documentation, once the package is installed, the shortName can be used with the dotnet new command:

```powershell
C:\templates> dotnet new edgamat-webapi
```

### Create a NuGet Package

Next we need to create a NuGet package which is what we will use to install the template. In the template folder, I created a `mytemplate.csproj` file with the following contents:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <PackageType>Template</PackageType>
    <PackageVersion>1.0</PackageVersion>
    <PackageId>Edgamat.MyTemplate</PackageId>
    <Title>Edgamat WebApi Solution Template</Title>
    <Authors>Matt Edgar</Authors>
    <Description>Template for .NET WebApi projects.</Description>
    <PackageTags>dotnet-new;templates</PackageTags>
    <TargetFramework>netstandard2.0</TargetFramework>

    <IncludeContentInPack>true</IncludeContentInPack>
    <IncludeBuildOutput>false</IncludeBuildOutput>
    <ContentTargetFolders>content</ContentTargetFolders>
  </PropertyGroup>

  <ItemGroup>
    <Content Include="**\*" Exclude="**\bin\**;**\obj\**" />
    <Compile Remove="**\*" />
  </ItemGroup>

</Project>
```

I then created the package:

```powershell
C:\templates> dotnet pack mytemplate.csproj
```

It generated the package here:

```powershell
C:\templates\mytemplate\bin\Release\Edgamat.MyTemplate.1.0.0.nupkg
```

One thing of note, when the package was built, there were 2 warning messages indicating the `.editorconfig` and `.gitignore` files were excluded. If I want to include these, I'll need to install and use the `nuget.exe` rather than the `dotnet` command line tool to create the package. 

### Installing the Package

To install the package, use the following command

```powershell
C:\templates> dotnet new install C:\templates\mytemplate\bin\Release\Edgamat.MyTemplate.1.0.0.nupkg
```

You can uninstall it using the following command:

```powershell
C:\templates> dotnet new uninstall Edgamat.MyTemplate
```

### The Results

I went to the command line and ran the following command:

```powershell
C:\templates> dotnet new edgamat-webapi -n test1
```

Here's what was created:

```
C:\templates\test1> ls

    Directory: C:\templates\test1

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----            3/4/2024  4:35 AM                src
d----            3/4/2024  4:35 AM                test
-a---            3/4/2024  4:35 AM             76 global.json
-a---            3/4/2024  4:35 AM            768 mytemplate.csproj
-a---            3/4/2024  4:35 AM           3235 mytemplate.sln
```

Not exactly what I was expecting. I wanted the files to have `mytemplate` replaced with the name (e.g. `test1`). Turns out I just missed a setting in the `mytemplate.csproj` file. I added this:

```json
  "sourceName": "mytemplate",
```

And now it produces the desired result:

```
C:\templates\test2> ls

    Directory: C:\templates\test2

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----            3/4/2024  4:41 AM                src
d----            3/4/2024  4:41 AM                test
-a---            3/4/2024  4:41 AM             76 global.json
-a---            3/4/2024  4:41 AM            768 test2.csproj
-a---            3/4/2024  4:41 AM           3235 test2.sln
```

There was a much better tutorial on the custom templates here:

[https://devblogs.microsoft.com/dotnet/how-to-create-your-own-templates-for-dotnet-new/](https://devblogs.microsoft.com/dotnet/how-to-create-your-own-templates-for-dotnet-new/)

### Summary

The process of creating a custom template is straightforward and I hope to make use of this moving forward. 
