---
layout: post
title: 'Remove and Sort Using in C#'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Code is a liability. The more code you have to maintain, the more challenging it becomes. Small bits of additional code can built up over time and it is important to stay on top of the waste as you build and change the code. A perfect case in point is removing unused and sorting using directives.

<!--more-->

## What are using directives?

In C#, using directives allow for the use of types outside of the current namespace without having to fully qualify the type:

```csharp
using System; // using directive

namespace MyProject
{
    public struct TransferData
    {
        public DateTime StartDate { get; set; } // Can use DateTime, rather than System.DateTime
    }
}
```

## Best Practices for maintaining using directives

Over time, the using directives can build up and you can have quite a few declared at the top of a class file. It is important to keep this list as clean as possible. The _de facto_ best practice advice is to ensure that any using directives no longer used by the code in the file should be removed. In addition, the using directives should also be sorted.

## Exceptions to the Remove Rule?

Some folks feel that removing all using directives when they aren't used is overkill. They prefer to always include a small set of using directives regardless if the file uses them or not. For example, some argue that the following should be included on all code files:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
```

the idea is that the majority of C# classes will end up using one or more of these and it says time to have them always available.

It is my opinion that this mindset is driven by years of having these inserted into new class files by Visual Studio's built-in templates. I was of this mindset for years. However, the lead developer on a new project asked me to switch to the 'remove all' rule for awhile and see if it still bothered me. I humbly admit that after a few weeks, I didn't notice the difference. The new IDE functionality to auto-insert using directives when you add a reference to a new type is so seamless these days that it becomes a non-issue. 

## Sort Usings Alphabetically or `System` First

Visual Studio (and JetBrains Rider) have options for how to sort using directives. You can choose to sort them all alphabetically or you can have all the System directives first followed by all others. Each group is sorted alphabetically.

```csharp
// Alphabetically
using MyProject.Business;
using MyProject.Data;
using System;
using System.Collections.Generic;
```

```csharp
// System First
using System;
using System.Collections.Generic;
using MyProject.Business;
using MyProject.Data;
```

I prefer the 'System' first approach. However, I have used the Alphabetically approach as well and it is not a significant difference to get fussed about. Having all developers follow a consistent standard is more important than the specifica standard they are following. 

## Making it easy

In Visual Studio, you can Remove and Sort using directives with a keyboard shortcut: CTRL+R, CTRL+G. 

In JetBrains Rider, you can Remove and Sort using directives with a keyboard shortcut: CRTL+ALT+O.

## Summary

Remove and Sort your using directives. It reduces the amount of code you need to maintain and makes the code easier to read and understand. 







