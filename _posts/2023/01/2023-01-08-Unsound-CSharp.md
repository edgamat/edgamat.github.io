---
layout: post
title: 'Unsound C#'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Recently the team I work with has debated the unsound behaviors of TypeScript. It got me thinking about what that means with respect to C#. Let's explore.

<!--more-->

### Sound versus Unsound 

Here's a good definition of soundness with respect to a programming language:

> Roughly speaking, a language is "sound" if the static type of every symbol is guaranteed to be compatible with its run-time value.

Source: https://effectivetypescript.com/2021/05/06/unsoundness/

Here's an example. If the TypeScript language service defines a variable to be a number at build time then it is guaranteed to be a number at run-time. But in TypeScript we know that may not be true.

```typescript
const xs = [0, 1, 2];  // type is number[]
const x = xs[3];  // type is number

console.log(x); // undefined
```

Source: https://effectivetypescript.com/2021/05/06/unsoundness/

The compiler says `x` is a number but at run-time it is `undefined`. This is an example of unsound behavior. 

Most programming languages are not 100% sound. Some have more unsound behaviors than others. Each language deals with this in their own way. TypeScript does not proclaim to be 100% sound, in fact it has explicitly made decisions to be unsound:

https://www.typescriptlang.org/docs/handbook/type-compatibility.html#a-note-on-soundness

Let's take a look then at C# and how it exhibits unsound behaviors.

### Unsoundness in C#

The `dynamic` type bypasses type checking and is inherently unsound. You have no guarantees at run-time about that a dynamic object holds:

```csharp
dynamic foo = new
{
    property = "Hello, World!"
};

foo.bar(); // Unhandled exception '<>f__AnonymousType0<string>' does not contain a definition for 'bar'
```

This compiles but does not work at run-time.

Anytime you have the compiler saying one thing and the run-time saying another you've got unsound behavior.

Here is another example using enumerations:

```csharp
enum Color 
{
  Blue = 0,
  Yellow = 1
}

var color1 = Color.Blue; // As expected
var color2 = (Color)99; // ???
```

`color2` contains the value 99, which doesn't raise a compile-time warning/error nor does it raise a run-time exception. This demonstrates that enum values are unsound and they should always be explicitly checked in code.

### Null Reference Types

Prior to C# 8, the compiler would not be aware of null reference types. As such you would likely see `NullReferenceException` occurrences at run-time. 

```csharp
string directoryName = System.IO.Path.GetDirectoryName("C:\\"); // directoryName = null

Console.WriteLine(directoryName.Length); // NullReferenceException: Object reference not set to an instance of an object. 
```

This is an example of unsound behavior. The compiler says directoryName is a string, but it is possible for it to contain `null`.

The inability to differentiate between a valid object and null has been a huge part of the history of C# (and other languages). Programs usually require additional code to deal with this possibility:

```csharp
string directoryName = System.IO.Path.GetDirectoryName("C:\\");

var directoryNameLength = 0;
if (directoryName != null)
{
  directoryNameLength = directoryName.Length;
}

Console.WriteLine(directoryNameLength);
```

This makes the code less concise and less readable. Language features have been added to make it 'safe' to access properties that may be null:

```csharp
string directoryName = System.IO.Path.GetDirectoryName("C:\\"); // directoryName = null

Console.WriteLine(directoryName?.Length ?? 0); // prints '0' when directoryName is null
```

The intention of this new syntax was to allow programs to more concisely deal with the possibility of a null value. It is debateable if this makes the program easier to read.

### Nullable Reference Types

C# 8 introduced a new compiler option that addresses these null reference issues. Enabling the 'nullable' option allows the compiler to treat a null reference type as a separate type from a non null reference type. This is how nullable value types have always been dealt with in C#.

https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references

Now the compiler will warn you when you are referring to the members of an object it thinks is `null` OR might be `null` if it can't make that determination. Here are a couple of statements that generate compiler warnings when the nullable compiler option is enabled:

```csharp
string foo = null;
// CS8600 Converting null literal or possible null value to non-nullable type.

var directoryName = Path.GetDirectoryName("C:\\");

Console.WriteLine(directoryName.Length);
// CS8602 Dereference of a possibly null reference.
```

There's a great reference for all these new compiler warnings:  
https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/nullable-warnings

These warnings help identify unsound behavior with respect to null reference types. If you heed these warnings and fix them, you may completely remove the possibility of having a NullReferenceException occur at run-time (one can dream, can't one?).

If you want you can treat these warnings as errors when the code is compiling, add the following to your CSPROJ project file:

```xml
  <PropertyGroup>
    <WarningsAsErrors>$(WarningsAsErrors);NU1605;CS8600;CS8602;CS8603</WarningsAsErrors>
  </PropertyGroup>
```

`CS8600;CS8602;CS8603` are the nullable warnings. `$(WarningsAsErrors);NU1605;` is the default for new C # projects. 

**NOTE** There can be a few false positives that show up. C# 10 introduced improvements to it's static analysis to reduce these false positives. 

Treating these as errors forces you to deal with them at compiler time (type checks, unit tests) rather than at run-time (unhappy users).

### I know more than the compiler

Sometimes you may feel you know more than the compiler. You _know_ that a reference type object will never be null. If you wish, you can override the compiler and force it to compile using the null-forgiving operator `!`. For example, this won't compile if you treat CS8603 as an error:

```csharp
private static string MyGetDirectoryName(string path)
{
    string? directoryName = Path.GetDirectoryName(path);

    return directoryName; // CS8603 Possible null reference return.
}
```

But by adding the null-forgiving operator to the return statement, it will:

```csharp
private static string MyGetDirectoryName(string path)
{
    string? directoryName = Path.GetDirectoryName(path);

    return directoryName!; // no longer an error
}
```

This is an example of unsound behavior. The compiler is happy to treat the return value as a string (not null). But at run-time, it could definitely throw an exception.

### Summary

We have looked at a couple of the unsound behaviors in C#, most notably, the problems with nullable reference types. As programmers we have to be aware of these possibilities and handle them accordingly. The improvements with how C# handles nullable reference types makes it much more difficult to get into trouble with nulls but it still gives you the opportunity to override that behavior and write unsound code. With any great power comes great responsibility. Use it wisely.

