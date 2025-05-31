---
layout: post
title: 'Inheritance Bugs in .NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

This week a curious bug occurred in our production environment. It ended up being caused by a change
in the inheritance chain for the compiled code. A recompile of the code fixed the issue, but I wanted
to dig into it a bit further.

<!--more-->

### The Context

We have a DLL that provides some custom functionality in a .NET application. This DLL gets dropped into
the folder where the EXE and other DLLs exist for the application. The custom DLL inherits several
of the classes in the application's DLLs and implements overrides of virtual methods.

For example, the application has a DLL containing a Widget class with a virtual method for determining
the title for a report the application generates:

```csharp
public class Widget
{
    public Widget()
    {
    }

    public virtual string GetReportTitle()
    {
        return "Monthly Widget Report";
    }
}
```

In the same DLL, there is another class that inherits from the Widget class:

```csharp
public class CustomWidget : Widget
{
    public CustomWidget()
    {
    }
}
```

This class does not override the base `GetReportTitle` method.

Our project references the `CustomWidget` DLL and implements a custom report title that includes the current date:

```csharp
public class OurCustomWidget : CustomWidget
{
    public OurCustomWidget()
    {
    }
    
    public override string GetReportTitle()
    {
        return $"{base.GetReportTitle()} for {DateTime.Now}";
    }
}
```

### The problem

One day the application and it's DLLs were updated. Our widget's report title in the application changed
from:

```text
Monthly Widget Report for 05/15/2025 11:00:12 AM
```

to

```text
for 05/15/2025 11:00:12 AM
```

If we switched back to the older DLLs the report title would appear correctly again. Switching back
to the new DLLs would cause the same incorrect report title, missing the 'base' title text. We knew
the new DLLs were the problem, but we didn't know why.

When we looked at the new code, we discovered that a new override had been created in the `CustomWidget` class:

```csharp
public class Widget
{
    public Widget()
    {
    }

    public virtual string GetReportTitle()
    {
        return "";
    }
}

public class CustomWidget : Widget
{
    public CustomWidget()
    {
    }

    public override string GetReportTitle()
    {
        return "Monthly Widget Report";
    }
}
```

The report title text was now being generated in the `CustomWidget` class, not the `Widget` class. We were
a bit confused. Our DLL was inheriting from `CustomWidget`, why wouldn't it use the new override?

### Early Binding in .NET

Our DLL was compiled against the original DLLs where the `GetReportTitle` was only implemented in
the `Widget` class. Even though the `OurCustomWidget` class inherits from `CustomWidget`, the
`base.GetReportTitle()` call is bound at compile time to the `Widget.GetReportTitle()` implementation.

This is called 'early-binding` or 'static-binding' in .NET. For many good reasons (performance being
the primary), the compiler will resolve to call the base method when it knows it is the only one implemented.
You can see this in the Intermediate Language (IL). This is the IL for the `GetReportTitle()`
override in the `OurCustomWidget` class:

```text
 IL_0007: call         instance string [Widgets]Widgets.Widget::GetReportTitle()
```

The compiler knows the `CustomWidget` doesn't have an override, so it calls the `Widget` method
directly.

And here lies our problem. When the new override was added to the `CustomWidget` class, our compiled
DLL didn't know it was there and continued to call the `Widget` method, which is now returning an
empty string.

The solution to the problem was simple: Recompile our code against the new DLLs. The compiler discovers
the new override in the `CustomWidget` class and binds to it, instead of the one in the `Widget` class.

### Is Early Detection Possible?

Thankfully we had someone who knew of the new override and brought it to our attention. It saved us
a lot of time diagnosing the cause and getting it fixed. But now I am wondering if it would be possible
to detect this type of change?

I think the answer is yes, but it would be fairly challenging. You would need to inspect the DLLs
before the change and after the change to see if there are any difference in the overrides.

You could scan each of the dependency DLLs (the ones that may change) and capture all the override
methods. You could then capture all the override methods in the dependent DLLs (our custom code).
You could then match them up and producer a snapshot file that indicates when overrides are available
for each method you are overriding.

If you extract the same data after the DLLs have changed, you should be able to detect if any override
methods (relating to your overrides) have been added/removed. This could then prompt you to investigate
if a recompile of your DLLs is necessary.

### Summary

Providing custom functionality using inheritance is very useful but it doesn't come without its problems
when the code is evolving independently. Care must be taken when introducing new overrides to ensure
existing code doesn't break. But once you know what to look for, it can be as easy as recompiling
your code to get things working correctly.
