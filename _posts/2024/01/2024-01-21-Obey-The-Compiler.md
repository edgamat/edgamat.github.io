---
layout: post
title: 'Obey the Complier'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I think it is very useful to treat C# compiler warnings as errors. Let's explore that idea! 

<!--more-->

When compiling a C# application, the compiler will let you know if something is a potential problem. It does this through warnings.

For example, let's build this code:

```csharp
var bar = Obey.Foo.Bar();

Console.WriteLine(bar.ToLower());

namespace Obey
{
    public static class Foo
    {
        public static string? Bar()
        {
            return Guid.NewGuid().ToString();
        }
    }
}
```

The return type of the `Bar` function is `string?`. That means the compiler thinks that the variable `bar` could be `null`. When you build this program,
the compiler warns the user that using the `ToLower()` method might fail:

```powershell
C:\Users\edgam\projects\Obey> dotnet build 
C:\Users\edgam\projects\Obey\Program.cs(3,19): warning CS8602: Dereference of a possibly null reference. [C:\Users\edgam\projects\Obey\Obey.csproj]
  Obey -> C:\Users\edgam\projects\Obey\bin\Debug\net8.0\Obey.dll

Build succeeded.
```

But as the final message indicates, it still builds the application. It is warning you of a runtime problem, not a compilation issue.

## Treat Warnings as Errors

Many times over the course of my career I have come across a C# project and upon building it, I am greeted with numerous (sometimes thousands!) warnings from the compiler.
It is incredibly disheartening when you see this. How can I tell which of these is a true issue or not? Which of these are due to me not setting up the code correctly on my PC? Which of these can I ignore? If I make a change to the code that generates a new warning, how will I ever find it in all that noise?

The experiences have motivated me to treat all warnings as errors. I've seen many developers use Visual Studio and due to the way it is configured, they never even see the warnings! By treating them all as errors it forces developers to deal with them.

But why, you might ask, should I bother? If the compiler only thinks of them as warnings, are they not that important? And this is where the trouble begins. As the example above demostrates, most warnings are highlighting a potential _runtime_ problem. It is importnat to investigate them all to be sure your code is of the upmost quality when it is running.

Treating warnings as errors means different things to differnt people. One person on your team might see an warning as something that needs to be fixed and another person might not. When reviewing a pull request, it is next to impossible to spot potential compiler warnings. If you force all warnings to be treated as errors, you know that if the code compiles there are no warnings that have been ignored. 

To enable this in .NET, add the `TreatWarningsAsErrors` element to your .csproj file:

```xml
<TreatWarningsAsErrors>true</TreatWarningsAsErrors>
```

## Be Practical

Obviously there will be times, either due to practical concerns or security issues, that you may need to leave the warnings as-is. In these cases, it makes sense to explicitly NOT treat certain warnings as errors. The C# compiler allows you to do this in one of two ways. First, you can add the following to your .csproj file:

```xml
<WarningsNotAsErrors>warningnumber1,warningnumber2</WarningsNotAsErrors>
```

You can place whatever warning codes you want in the list and they will no longer be treaed as errors.

Another way to accomplish the same goal for all projects in a solution is to use a `Directory.Build.props` file:

```xml
<Project>
  <PropertyGroup>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>
</Project>
```

Sometimes you may not want to warn users of the issues at all:

```xml
<NoWarn>warningnumber1,warningnumber2</NoWarn>
```

This can be very useful when certain warnings are something that cannot be 'fixed' (ie. false positives). Being constantly warned about them can be distracting. While this is potentailly dangerous if you start ignoring issues that could cause you problems, it is a useful option to have. For example, in one project, we were using a third-party DLL that wasn't compatible with the current version of .NET we were using. But we had fully tested the portion of the functionality we required and it worked perfectly. Being constantly warned of the potential incompatibility was not necessary and we used the NoWarn to remove the warnings from the compilation output.

## Summary

I recommend all projects treat all warnings as errors. Only if they are investigated and deemed safe to ignore should they be treated as warnings. This approach makes those decisions explicit (as it would be something you would need to explicitly codify in a pull request / git commit) and much easier for the team to understand.
