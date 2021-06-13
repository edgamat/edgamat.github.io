---
layout: post
title: 'DateOnly/TimeOnly Types in .NET 6'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In the latest preview of .NET 6.0, two new struct types were introduced: `DateOnly` and `TimeOnly`. They fill a gap for use cases long asked for in .NET.

<!--more-->

### DateOnly

The `DateTime` struct has its limitations when modeling real-life business scenarios, must notably, when you require just the date or just the time. Take a person's birth day for example. The time of day is typically not applicable. But when we only have the `DateTime` struct to store such data, you would simply leave the time off:

```csharp
var birthDay = new DateTime(1980, 3, 28);
```

However, the `DateTime` still requires a time component, which would be midnight by default. Since this is a `DateTime` value, it can be manipulated with TimeZone changes and the `DateTimeKind` of the instance can be `Unspecified`, `Local` or `UTC`. None of this matters to a 'date' which represents all the points in time on the single day. 

I have had many instances when this has caused bugs when the 'date' gets shifted by the current time zone offset and can end up being incorrectly stored in the database or displayed to the user.

Now with .NET 6, it is possible to represent the date without concern for the time:

```csharp
var birthDay = new DateOnly(1980, 3, 28);
```

And since this new struct doesn't have the concepts of timezones or 'kind', these issues are no longer present.

The new `DateOnly` struct API is very robust and provides a lot of useful functions:

```csharp
// Convert from a DateTime value
DateOnly today = DateOnly.FromDateTime(DateTime.Today);

// Add by day month or year
var oneWeekFromNow = today.AddDays(7);
var oneMonthFromNow = today.AddMonths(1);
var oneYearFromNow = today.AddYears(1);

// Combine with time
var supperTime = today.ToDateTime(new TimeOnly(17, 0));

// Parsing
var forthOfJuly = DateOnly.Parse("2021-07-04");
```
### TimeOnly

The new `TimeOnly` struct represents an instance of time in a given day. A typical use case would be a recurring appointment or meeting, or when you get up each morning or go to bed. The 'date' isn't important. When using the `DateTime` to represent such a concept, you would typically choose an arbitrary date (usually 01/01/0001). The `TimeOnly` struct now allows the code to more accurately represent the time and avoid bugs due to time zone conversions and other manipulations of `DateTime` instances that don't apply to just the 'time'.

It too has the expected API:

```csharp
var myBedTime = new TimeOnly(21, 0, 0);

var whenIWakeUp = new TimeOnly(4, 0, 0);

// This correctly calculates 7 hours.... something DateTime was terrible at!
var sleep = whenIWakeUp - myBedTime; 

var now = TimeOnly.FromDateTime(DateTime.Now);
var laterToday = now.AddMinutes(90);
var laterTomorrow = now.AddHours(25);
```

Curious, there is no `AddSeconds()` method.

I found this one interesting:

```csharp
var elapsedTime = TimeSpan.FromHours(80);

var whenItStarted = new TimeOnly(9, 0);

var whenItEnded = whenItStarted.Add(elapsedTime, out var wrappedDays);
```

According to the documentation, `wrappedDays` represents the number of 'circulated days':

> When this method returns, contains the number of circulated days resulted from this addition operation.

In the above example, `wrappedDays` equals 3. From 9 AM to 80 hours later, 3 days have gone by. Now sure when this would be useful, but I know some someone will cheer at having this capability. :-)

### Summary

I love these new additions and hope folks will find them as useful as I do. Checkout Matt Johnson-Pint's blog post for more details on the new features:

[https://devblogs.microsoft.com/dotnet/date-time-and-time-zone-enhancements-in-net-6/](https://devblogs.microsoft.com/dotnet/date-time-and-time-zone-enhancements-in-net-6/)

