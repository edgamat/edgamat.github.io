---
layout: post
title: 'Time is the Enemy When Testing .NET'
author: 'Matthew Edgar'
pub_date: '2019-09-14'
excerpt_separator: <!--more-->
---

If your application uses time, then it is necessary for you to consider how your application thinks about time. It can be
an enemy if you let it, but it can be a friend if you use it right.

<!--more-->

I'm not going to attempt to tell you that using `DateTime.Now` is evil (in most cases it is) nor am I
going to promote alternatives like [NodaTime][noda] (although that will be the topic of another post). No, I want
you to think about time when testing your code.

Consider the following example. This function is responsible for determining if an action is enabled based
on the current time (say, Christmas Day):

```csharp
public bool IsEnabled()
{
    var launchDate = new DateTime(2019, 12, 25, 0, 0, 0, DateTimeKind.Utc);

    return DateTime.UtcNow > launchDate;
}
```

This function is nearly impossible to write a unit test for because the call to `DateTime.UtcNow` depends on the
current system time of the machine running the code.

However, let's say you change it to be:

```csharp
public bool IsEnabled(DateTime utcNow)
{
    var launchDate = new DateTime(2019, 12, 25, 0, 0, 0, DateTimeKind.Utc);

    return utcNow > launchDate;
}
```

That's better. Now at least you can change when 'Now' is and write unit tests. As a general rule,
it is better to make the current system date a dependency to your code, rather than calculating
it. In this example, we chose to pass it in as an input parameter.

By the way, did you notice the subtle bug that was introduced? `DateTime.UtcNow` has a `DateTimeKind` of `DateTimeKind.Utc`.
But what of the input parameter `utcNow`? Without checking, we don't know. So we really should do this:

```csharp
public bool IsEnabled(DateTime utcNow)
{
    if (utcNow.Kind != DateTimeKind.Utc) throw new ArgumentException($"Not provided in UTC", nameof(utcNow));

    var launchDate = new DateTime(2019, 12, 25, 0, 0, 0, DateTimeKind.Utc);

    return utcNow > launchDate;
}
```

Finally, one more alternative would be to pass in an object that is responsible for calculating the time:

```csharp
public bool IsEnabled(ITimeProvider timeProvider)
{
    var launchDate = new DateTime(2019, 12, 25, 0, 0, 0, DateTimeKind.Utc);

    var utcNow = timeProvider.UtcNow;

    return utcNow > launchDate;
}
```

In my opinion, this is a more desirable approach. You get the benefit of being able to inject any time value
you wish, and you can ensure the value is in the form you require (that is, the time is provided in UTC).

### Delaying the Inevitable

When your code needs to pause for a time delay, you can code something like this:

```csharp
public Task ExecuteAsync(CancellationToken token)
{
    do while (!token.IsCancellationRequested)
    {
        /// Do work

        // Wait for one minute and do some more work
        await Task.Delay(TimeSpan.FromMinutes(1), token);
    }
}
```

This is not difficult to test, but the delays are going to slow down your testing since it
will wait 60 seconds. It is more testable to pass in an object to provide the delay:

```csharp
public Task ExecuteAsync(IDelayProvider delayProvider, CancellationToken token)
{
    do while (!token.IsCancellationRequested)
    {
        /// Do work

        // Wait for one minute and do some more work
        await delayProvider.DelayAsync(TimeSpan.FromMinutes(1), token);
    }
}

public class DelayProvider : IDelayProvider
{
    public Task DelayAsync(TimeSpan timeSpan, CancellationToken token)
    {
        return Task.Delay(timeSpan, token);
    }
}
```

Then for testing, you can provide a fake provider that has no delay, regardless of the time span provided:

```csharp
public class FakeDelayProvider : IDelayProvider
{
    public Task DelayAsync(TimeSpan timeSpan, CancellationToken token)
    {
        return Task.CompletedTask;
    }
}
```

I hope this demonstrates how dealing with Time in your application can impact how you test your code. If
you do a bit of planning and design work, you can make a testable code base, even if you are dealing with Time.

Oh, and yes `DateTime.Now` is evil. Don't forget.

[noda]: https://nodatime.org
