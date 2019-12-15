---
layout: post
title: 'Using NodaTime with Entity Framework Core 3'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I have built many C# applications that used dates and times as part of the core domain logic. I
never liked the fact that dates (like birth dates) had to include a time component. For that
reason and a few others, I'm going to try using the date and time concepts from the NodaTime
library and see if things improve.

<!--more-->

## What is NodaTime?

[NodaTime][nodatime] is an alternative date and time library for .NET applications. It is
a port of the JodaTime library from Java, but with some API differences which make it better
suited for .NET applications.

The online documentation provides a great explanation for why the library exists. In my situation,
I want to use it to help me better express dates and times in my applications.

## Motivation

I have found a few cases over the years where the built-in `DateTime` type fails to model
things the way I'd hope:

- Dates without time,
- Future dates and times,
- Persistence/serialization

## Accept the Dependency?

Adding an external dependency to a domain model is an important decision. Keeping these
external dependencies to a minimum is usually desirable. How does this apply to adding
the NodaTime library?

Essentially, using the NodaTime types replaces using the `DateTime` type. If the domain
models date and time using `DateTime` you should consider directly swapping the
`DateTime` uses with the appropriate NodaTime types.

## Common Concepts

Here are some ways to use the concepts in NodaTime.

### Now

NodaTime has a struct called `Instant` that represents an instant on the global timeline. Use
the following to get an instant in time:

```csharp
Instant now = SystemClock.Instance.GetCurrentInstant();
```

An Instant is the number of nanoseconds which have occurred since the Unix epoch (midnight at the start of January 1st 1970, UTC).

To convert it to a time zone is the purpose of the `ZonedDateTime`:

```csharp
ZonedDateTime nowInIsoUtc = now.InUtc();
```

[nodatime]: https://nodatime.org
