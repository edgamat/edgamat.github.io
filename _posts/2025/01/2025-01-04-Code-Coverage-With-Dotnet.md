---
layout: post
title: 'Generating Code Coverage Reports .NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Code coverage can be a highly debatable topic, but I find it very helpful in knowing which classes
are under represented in my suite of unit tests. I've relied on the UI versions of test runners to
provide this data. Let's explore what we can do with the command line.

<!--more-->

## Code Coverage

We write unit tests in order to make the growth of our code base more sustainable. One measure of how
well our unit tests cover our code base is called code coverage. It is a measure of how many lines
of our code, or branches in the code, are represented in our test cases. It is provided as a percentage
Zero means there are no tests that exercise the code in a file, 100 means that all lines/branches in
the file are exercised by one or more tests.

It is debatable which percentage threshold is what teams should strive for. Some advocate that 100%
coverage is the goal. I don't think that is wise. There are some lines of code that don't require
unit tests. If there is a bug in these lines, the program would simply not work (a bug would be so
obvious no one would miss it). So if 100% is not the goal, what is? I think it depends on the code
base, the project the team is working on, and the team composition. But in general, as a starting point,
I start with a threshold of 80% and adjust from there.

I used JetBrains "dotCover" for several years to provide the code coverage metrics on my .NET projects.
The project I am currently working on is using Visual Studio within virtual machines, so I have moved away
from using JetBrains tools. I must admit that means I have not focused as much on code coverage as I
used to. I wanted to explore what I could do from the command line.

In our CI/CD pipeline we are generating code coverage metrics using the capabilities in the `dotnet test`
test runner. This is what I wanted to research a bit more and see what I could setup to view the code
coverage data in my local workspace.

## Command Line Code Coverage

To enable the code coverage capabilities using the test runner, I first add a NuGet package to
each of the unit test projects:

```powershell
dotnet add package coverlet.collector
```

Now I can run the tests and generate coverage data:

```powershell
dotnet test --collect:"XPlat Code Coverage"
```

This will generate a new folder called `TestResults` where there are XML files that contain the test
results. The Microsoft docs go into much detail about these files:

[https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage?tabs=windows](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage?tabs=windows)

Now that I have the results I want to generate a report of their findings. In the past I worked on
a TypeScript Node.js project. We used Istanbul's `nyc` command line tool to generate a test-based
report that developers could use. I want to see if I can do something similar with .NET.

I am going to try using the de facto report generator, which I will install as a global tool:

```powershell
dotnet tool install --global dotnet-reportgenerator-globaltool
```

Here is the project's website/repository:  
[https://github.com/danielpalme/ReportGenerator](https://github.com/danielpalme/ReportGenerator)

Now I can generate a report:

```powershell
reportgenerator -reports:**/TestResults/**/coverage.cobertura.xml -targetdir:CoverageReport
```

Here are the results:

```text
2025-01-05T05:46:12: Arguments
2025-01-05T05:46:12:  -reports:**/TestResults/**/coverage.cobertura.xml
2025-01-05T05:46:12:  -targetdir:CoverageReport
2025-01-05T05:46:12: Writing report file 'CoverageReport\index.html'
2025-01-05T05:46:12: Report generation took 0.2 seconds
```

As you can see, it generates an HTML report by default. To override this behavior, you explicitly
set the report types using a command line parameter:

```powershell
reportgenerator -reports:**/TestResults/**/coverage.cobertura.xml -targetdir:CoverageReport -reporttypes:"Html;TextSummary"
```

Now it will generate the original HTML details report and a text summary. The HTML report can be useful for seeing
which files are under represented by the suite of unit tests. The text summary gives you a quick way
to see a summary of the same data:

```powershell
cat CoverageReport\summary.txt
Summary
  Generated on: 1/5/2025 - 5:52:24 AM
  Coverage date: 1/5/2025 - 5:46:05 AM
  Parser: MultiReport (2x Cobertura)
  Assemblies: 1
  Classes: 1
  Files: 1
  Line coverage: 100%
  Covered lines: 6
  Uncovered lines: 0
  Coverable lines: 6
  Total lines: 14
  Covered branches: 0
  Total branches: 0
  Method coverage: 100% (2 of 2)
  Full method coverage: 100% (2 of 2)
  Covered methods: 2
  Fully covered methods: 2
  Total methods: 2

CodeCoverageLib              100%
  CodeCoverageLib.MathLib    100%
```

## Minimum Coverage Thresholds

The report generator has many settings, including minimum coverage thresholds:

Command line parameter | Explanation
--- | ---
`minimumCoverageThresholds:lineCoverage=null` | Threshold for minimum line coverage. If line coverage falls below this threshold, _ReportGenerator_ will exit unsuccessfully. Value has to be a number (percentage) between 1 and 100.
`minimumCoverageThresholds:branchCoverage=null` | Threshold for minimum branch coverage. If branch coverage falls below this threshold, _ReportGenerator_ will exit unsuccessfully. Value has to be a number (percentage) between 1 and 100.
`minimumCoverageThresholds:methodCoverage=null` | Threshold for minimum method coverage. If method coverage falls below this threshold, _ReportGenerator_ will exit unsuccessfully. Value has to be a number (percentage) between 1 and 100.
`minimumCoverageThresholds:fullMethodCoverage=null` | Threshold for minimum full method coverage. If full method coverage falls below this threshold, _ReportGenerator_ will exit unsuccessfully. Value has to be a number (percentage) between 1 and 100.

Here is where you can find more about the settings for the report generator:  
[https://github.com/danielpalme/ReportGenerator/wiki/Settings](https://github.com/danielpalme/ReportGenerator/wiki/Settings)

With these parameters, the report generator will fail if these minimum thresholds are not met. Here
is a sample where the minimum line coverage threshold is 80%:

```powershell
reportgenerator -reports:**/TestResults/**/coverage.cobertura.xml -targetdir:CoverageReport -reporttypes:"Html;TextSummary" minimumCoverageThresholds:lineCoverage=80
```

## Final Script

I now have a script I can run from the command line when I want to see my code coverage data:

```powershell
# dotnet add package coverlet.collector
# dotnet tool install --global dotnet-reportgenerator-globaltool
dotnet test --collect:"XPlat Code Coverage"
reportgenerator -reports:**/TestResults/**/coverage.cobertura.xml -targetdir:CoverageReport -reporttypes:"Html;TextSummary" minimumCoverageThresholds:lineCoverage=80
cat CoverageReport/Summary.txt
```

## Summary

Code coverage is a useful tool in gauging the health/usefulness of your quite of unit tests. Regardless
of which minimum threshold you are striving to achieve, seeing the current code coverage metrics is
very helpful. I prefer the command line for such things as if is usually very fast and something I can
create a script for. I am pleased I can do this with the .NET test runner.
