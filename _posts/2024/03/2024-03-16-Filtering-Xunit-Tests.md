---
layout: post
title: 'Filtering xUnit Tests in .NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When running xUnit tests, sometimes you don't want to run all of them. You can use filters to run a subset of the tests from the command line.

<!--more-->

### The Problem

In addition to running unit tests, we use xUnit to run integration tests. These tests are stored in separate projects. For example:

```
.\tests\IntegrationTests.MyApp.Core\IntegrationTests.MyApp.Core.csproj
.\tests\UnitTests.MyApp.Core\UnitTests.MyApp.Core.csproj
```

By default, if you run the `dotnet test` command, it will discover all the tests from both projects and run them.

But sometimes, you don't want to run all of them. Sometimes you want to run just the unit tests or just the integration tests. Sometimes you want to only run the tests for a specific class. Sometimes you only want to run a single test. Let's look at how this can be done.

### Running Test from the Command Line

As with most things in .NET, I prefer to run tests using the command line:

```
dotnet test
```

Without any arguments, this command will find all tests cases in all assemblies and run them. To only run the tests from a single assembly, you have a couple of options. One option is to change your directory to the project folder and then run the tests:

```powershell
cd .\tests\UnitTests.MyApp.Core
dotnet test
```

The second option is to provide the test runner with a path to the tests:

```powershell
dotnet test .\tests\UnitTests.MyApp.Core
```

### Introducing Traits

Rather than using projects to organize tests, xUnit provides a means of organizing tests using metadata attributes called **traits**. A trait can be applied at the assembly, class or test method level. To keep things simple, let's assume that all tests will be assigned a trait called "TestCategory". To assign all test cases in an assembly a test category of "Integration", create a file (the name of the file is not important, but I like using `AssemblyInfo.cs`) with the following:

```csharp
[assembly: AssemblyTrait("TestCategory", "Integration")]
```

You can apply multiple traits if you wish:

```csharp
[assembly: AssemblyTrait("TestCategory", "Unit")]
[assembly: AssemblyTrait("TestCategory", "Domain")]
```

To assign a trait at the class or method level, using the `Trait` attribute:

```csharp
[Trait("TestCategory", "Math")]
public class MathWorker
{
    [Fact]
    [Trait("TestCategory", "Addition")]
    public void ShouldAddTwoNumbers()
    {
        MathWorker.Add(1, 2).Should().Be(3);
    }
}
```

With these traits in place, you can now apply filters to control which tests are run via the command line:

```powershell
dotnet test --filter "TestCategory = Unit"
```

You can use multiple traits too:

```powershell
# Tests with a TestCategory traits of both Unit and Math
dotnet test --filter "(TestCategory = Unit) & (TestCategory = Math)"

# Tests with a TestCategory trait of either Unit and Math
dotnet test --filter "(TestCategory = Unit) | (TestCategory = Math)"
```

### Filter By Fully Qualified Names

The command line test runner also allows you to filter by the name of the test. The syntax is similar to the one used for traits:

```powershell
# Run a test with the specified full name (exact match)
dotnet test --filter "FullyQualifiedName=Namespace.ClassName.MethodName"

# Run tests that contain the specified name
dotnet test --filter "FullyQualifiedName~Namespace.ClassName"
```

For example, if I want to run only the "ShouldAddTwoNumbers" test in my sample code, I would use this filter:

```
dotnet test --filter "FullyQualifiedName=UnitTests.MyApp.Core.MathWorkerTests.ShouldAddTwoNumbers"
```

Or if I wanted to run all the tests in the MathWorkerTests class, I would use this filter:

```
dotnet test --filter "FullyQualifiedName~MathWorkerTests"
```

The filters that use the FullyQualifiedName syntax are very useful when you are working on a single test or a set of tests in a single file.

### Applying a Default Filter using Run Settings

The command line test runner can also use settings stored in a file to control the execution of the tests. There is a great description of this process here:

[https://learn.microsoft.com/en-us/visualstudio/test/configure-unit-tests-by-using-a-dot-runsettings-file](https://learn.microsoft.com/en-us/visualstudio/test/configure-unit-tests-by-using-a-dot-runsettings-file)

For filtering, you create a `.runsettings` file to store your filter:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RunSettings>
  <RunConfiguration>
    <TestCaseFilter>(TestCategory != Integration)</TestCaseFilter>
  </RunConfiguration>
</RunSettings>
```

To run the tests using these settings, use the -s option:

```powershell
dotnet test -s .\.runsettings
```

The major advantage of this settings file is with Visual Studio's test runner. It will automatically find and use the filter described in this file. We use this ability to ensure that when you run "all tests" in Visual Studio, it filters out all the integration tests and only runs the unit tests. Then in our deployment pipeline we can be selective about which tests we run via the command line:

```powershell
# During the commit phase we run the unit tests:
dotnet test --filter "(TestCategory != Integration)"

# During the acceptance phase we run the integration tests:
dotnet test --filter "(TestCategory = Integration)"
```

### Summary

The `dotnet test` command includes some valuable ways of filtering your tests. Using the xUnit trait attributes you can setup as much control over which tests you run as you require. Use the `.runsettings` file to establish a consistent set of default tests to run in Visual Studio.
