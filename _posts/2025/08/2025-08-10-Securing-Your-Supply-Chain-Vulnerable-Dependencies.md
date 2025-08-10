---
layout: post
title: 'Securing Your Supply Chain - Vulnerable Dependencies'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

<!--more-->

### Dangers Are Everywhere

Software developers have always needed to be on guard for possible attacks to the software we write. This is nothing new. But we often don't spend enough time looking at our supply chain for possible security breaches. A supply chain attack is when an attacker compromises something during the build or delivery of the software application. Malicious code ends up in the artifacts we deliver, not in the source code repository.

I primarily build .NET applications. There are several examples of supply chain attacks we must guard against:

- A direct NuGet package dependency that includes malicious code,
- An indirect, or transitive, NuGet package that is pulled in by another package,
- A build tool or SDK that pulls in malicious code at compile-time,
- A container image in the deployment that has been tampered with,
- The NuGet feed (i.s. artifact repository) that has been tampered with,
- A breach in the CI/CD pipeline, allowing attackers to inject malicious code.

In all these cases, there is an attack between the source code and the final delivered binary.

Let's look at the types of vulnerabilities that arise from the dependencies you use.

### Vulnerable Dependencies

Every time you add a new dependency to a project, you take on more risk to your supply chain. Therefore the first rule in protecting yourself is to not add any dependency that can be avoided. Sometimes we add dependencies out of convenience. Take for example, the AutoMapper NuGet package. It saves time during development by proving mapping functions transforming data objects from one form to another. This is done out of convenience, so you don't have to hand-craft such mapping functions, nor maintain them. But I have found that these mapping functions are typically trivial to write and we use AutoMapper far too much. Removing this package reduces your exposure to attacks from the dependencies you use.

While I am not saying you should remove AutoMapper everywhere, it illustrates the point that some dependencies we may be able to do away with. Only take on a dependency if you really need to. Sometimes hand-crafting the functionality may be worth it, from a security perspective.

It is also important to detect any vulnerable packages, both direct and transitive. The .NET CLI provides the tools to do this. Running this command will list all the packages your solution depends on:

```powershell
# .NET 9 or earlier
dotnet list package

# .NET 10
dotnet package list
```

But it can even be more useful. You can list any packages that are outdated:

```powershell
dotnet list package --outdated
```

This can be helpful in spotting newer versions of packages you depend on. Include the `--include-prerelease` option if you depend on pre-release packages.

More importantly from a security perspective, you can list any packages with known vulnerabilities:

```powershell
dotnet list package --vulnerable
```

This will scan the direct dependencies only. To scan all transitive dependencies, run this command:

```powershell
dotnet list package --vulnerable --include-transitive
```

You will get a report like this:

```powershell
Build succeeded in 4.3s

The following sources were used:
   https://api.nuget.org/v3/index.json
   C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\

Project `MyProject` has the following vulnerable packages
   [net8.0]:
   Transitive Package           Resolved   Severity   Advisory URL
   > System.Data.SqlClient      4.6.0      Moderate   https://github.com/advisories/GHSA-8g2p-5pqh-5jmc
                                           High       https://github.com/advisories/GHSA-98g6-xh36-x2p7
```

If you are not clear why you have a dependency on this package, you can investigate using the following command:

```powershell
dotnet nuget why System.Data.SqlClient
```

The report looks like this:

```powershell
Project 'MyProject' has the following dependency graph(s) for 'System.Data.SqlClient':

  [net8.0]
   │
   └─ DbUp (v4.6.0)
      └─ dbup-sqlserver (v4.6.0)
         └─ System.Data.SqlClient (v4.6.0)
```

Each of the packages identified as vulnerable will have an Advisory URL you can follow to view the type of vulnerability and any possible mitigation strategies. In this case, the advisory indicates that there is a newer version of the package that patches the issue. You may find that the direct dependency (in this case, the `dbup` package) has a newer version that uses the new version of the vulnerable package. If not, you can go into each of the affected projects in you solution and add a reference to the newer package:

```xml
<PackageReference Include="System.Data.SqlClient" Version="4.8.6" />
```

Obviously this depends on the types of changes (it may break things!) so it is important to test these newer packages with your code.

### Always On Guard

In our environment we use a CI/CD pipeline to build all the artifacts we deploy. In this pipeline we have a step that checks for vulnerable packages:

```bash
dotnet list package --vulnerable | tee vulnerable.log
```

The job checks the `vulnerable.log` file for any signs of a vulnerable package. If detected, it writes a warning to the pipeline; It does not fail the build. We felt it would be possible to have false-positives that would prevent us from deploying code changes. The warning allow us to detect the vulnerabilities and investigate, while still deploying the code.

In addition, we (currently) don't scan for vulnerable transitive packages. We felt if the vulnerabilities were serious enough, the package we directly rely on would release a fix. I think this is something we will be changing. I think it is safer to report all transient packages that are vulnerable and let us decide how best to resolve the issues they may represent in our context.

### Summary

Our supply chain poses a possible attack vector for those who wish to embed malicious code into the artifacts we build and deploy. One way to do that is through the dependencies that we use in our software. It is critical that we pay attention to these dependencies. We should eliminate any we don't truly need, and we should always be on the lookout for known vulnerabilities. The .NET CLI makes this possible through some very simple commands that can be added to your CI/CD pipeline and be checked automatically with every build.
