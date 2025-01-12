---
layout: post
title: 'Using Central Package Management with .NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

It is often the case that a solution with a large number of projects share a lot of the same NuGet packages.
Central Package Management is a means to help keep the package versions synchronized across the projects.

<!--more-->

## Central Package Management

Central Package Management is a new feature introduced with NuGet 6.2 that allows you to centrally
manage all the NuGet package dependencies for the projects in your solution. You create a
`Directory.Packages.props` file and use it to store all the packages you are using across all
your projects. Here is a sample:

```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
        <PackageVersion Include="{Package Name 1}" Version="{version 1}" />
        <PackageVersion Include="{Package Name 2}" Version="{version 2}" />
        ...
        <PackageVersion Include="{Package Name N}" Version="{version N}" />
  </ItemGroup>
</Project>
```

The `ManagePackageVersionsCentrally` MSBuild property indicates that the solution should use
the `Directory.Packages.props` to track the versions of all packages in one central location.

If you navigate to a project and add a package, a `PackageVersion` element will be added to the
`Directory.Packages.props` file and in the project's csproj file, a `PackageReference` is added
with only the name of the package provided:

```xml
<PackageReference Include="{Package Name}" />
```

Reference: [https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management][homepage]

## Updating an Existing Solution

### Step 1 - Find all the packages referenced in all the projects

It can be useful at the start to get a list of all the packages. You can use this list to see
which packages have different versions referenced in different projects.

**NOTE** Thanks to Milan Jovanovic for inspiring me to look into Central Package Management more deeply
and for providing a starting sample of this PowerShell script.

[https://www.milanjovanovic.tech/blog/central-package-management-in-net-simplify-nuget-dependencies](https://www.milanjovanovic.tech/blog/central-package-management-in-net-simplify-nuget-dependencies)

```powershell
# file: analyze-package-versions.ps1

# Scan all .csproj files and collect package references along with their projects
$packageReferences = Get-ChildItem -Filter *.csproj -Recurse | ForEach-Object {
    $projectPath = $_.FullName
    $projectName = $_.BaseName

    # Extract PackageReference information from the .csproj file
    Get-Content -Path $projectPath |
        Select-String -Pattern '<PackageReference Include="([^"]+)" Version="([^"]+)"' -AllMatches |
        ForEach-Object { $_.Matches } |
        ForEach-Object {
            @{
                Package = $_.Groups[1].Value
                Version = $_.Groups[2].Value
                Project = $projectName
            }
        }
}

$nonUniqueVersions = @()

# Group packages and format the output
$packageReferences | Group-Object -Property Package | ForEach-Object {

    # Group by version for each package
    $versions = $_.Group | Group-Object -Property Version

    # determine if all references to the package use the same version
    $uniqueVersions = $versions | Select-Object -ExpandProperty Name | Select-Object -Unique

    if ($uniqueVersions.Count -ne 1) {

        # keep track of non-unique versions
        $nonUniqueVersions += $_.Name

        Write-Host "$($_.Name)" -ForegroundColor Red  # Package name

        # If there are multiple versions, list each version with its projects
        $versions | ForEach-Object {
            $version = $_.Name
            $projects = $_.Group | Select-Object -ExpandProperty Project
            Write-Host "`t$version - $($projects -join ', ')" -ForegroundColor Red
        }
    }

    Write-Output ""  # Blank line between packages
}

if ($nonUniqueVersions.Count -eq 0) {
    Write-Host "All packages have consistent versions." -ForegroundColor Green
} else {
    Write-Host "One or more packages have inconsistent versions" -ForegroundColor Red
    exit 1
}
```

Running the script on my sample solution provides the following:

```text
 .\analyze-package-versions.ps1

Swashbuckle.AspNetCore
        6.6.2 - WebApplication2
        6.9.0 - WebApplication1

One or more packages have inconsistent versions
```

### Step 2 - Add the `Directory.Packages.props` file

When updating a solution to use Central Package Management, add an initial `Directory.Packages.props`
file in the solution root:

```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
  </ItemGroup>
</Project>
```

The next steps can be done manually, but I have written a PowerShell script to automate the process.

### Step 3 - Add package references

We need to scan the projects and add all the project references. Here is the script I wrote to perform that task:

```powershell
$solutionRoot = Get-Location

# Install NuGet.Versioning package to parse and compare semantic versions of the packages
nuget install NuGet.Versioning -Version 6.12.1 -OutputDirectory .\packages
$assemblyPath = ".\packages\nuget.versioning.6.12.1\lib\netstandard2.0\NuGet.Versioning.dll"
[System.Reflection.Assembly]::LoadFrom($assemblyPath)

# Get the latest semantic version from a list of versions
function Get-LatestSemanticVersion {
    param (
        [string[]]$versions
    )
    # Use NuGet.Versioning to parse and compare versions
    $versions | Sort-Object -Descending -Property {
        [NuGet.Versioning.NuGetVersion]::Parse($_)
    } | Select-Object -First 1
}

# Find all PackageReference elements in .csproj files and group them by package name
$packages = Get-ChildItem -Filter *.csproj -Recurse |
    Get-Content |
    Select-String -Pattern '<PackageReference Include="([^"]+)" Version="([^"]+)"' -AllMatches |
    ForEach-Object { $_.Matches } |
    Group-Object { $_.Groups[1].Value } |
    ForEach-Object {
        [PSCustomObject]@{
            Name = $_.Name
            # Find the most recent semantic version using the custom function
            Version = Get-LatestSemanticVersion -versions ($_.Group.ForEach({ $_.Groups[2].Value }) | Select-Object -Unique)
        }
    } |
    Sort-Object { $_.Name }

if ($packages.Count -eq 0) {
    Write-Host "No PackageReference elements found in .csproj files"
    exit
}

# Create the Directory.Packages.props file
$packageVersions = @(
    $packages | ForEach-Object {
        "    <PackageVersion Include=`"$($_.Name)`" Version=`"$($_.Version)`" />"
    }
    ) -join "`n    "

$propsFilePath = Join-Path $solutionRoot 'Directory.Packages.props'
$propsContent = @"
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    $packageVersions
  </ItemGroup>
</Project>
"@

# Write the props content to the file
Set-Content -Path $propsFilePath -Value $propsContent
Write-Host "Created $propsFilePath"
```

### Step 4 - Remove version attributes in all .csproj files

Lastly we need to go through and remove the version attributes from PackageReference elements in all .csproj files:

```powershell
Get-ChildItem -Filter *.csproj -Recurse |
    ForEach-Object {
        $csprojPath = $_.FullName
        $content = Get-Content -Path $csprojPath
        $updatedContent = $content -replace '<PackageReference Include="([^"]+)" Version="([^"]+)"',
            '<PackageReference Include="$1"'
        Set-Content -Path $csprojPath -Value $updatedContent
        Write-Host "Updated $csprojPath"
    }
```

## Don't Like PowerShell?

I wrote all this in PowerShell for a couple of reasons. One, I wanted to learn all the steps involved and two, I need to practice using PowerShell as often as I can.

However, there is a tool you can install that does all this for you:

[https://github.com/Webreaper/CentralisedPackageConverter](https://github.com/Webreaper/CentralisedPackageConverter)

```powershell
dotnet tool install CentralisedPackageConverter --global
central-pkg-converter path/to/solutionroot
```

It works very well and has some additional feature (like doing a dry run before making any changes, reverting back to non central package management references).

## Summary

Central Package Management is a great way to ensure all the projects in your solution are using the same version. Updating an existing solutions to use Central Package Management is not too difficult using the PowerShell scripts or the `central-pkg-converter` dotnet tool.

[homepage]: https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management
