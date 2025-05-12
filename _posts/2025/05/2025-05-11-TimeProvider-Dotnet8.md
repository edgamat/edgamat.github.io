---
layout: post
title: 'Using the new TimeProvider in .NET 8'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I have created many custom implementations of a time abstraction to make it easier to test code
involving the `DateTime` or `DateTimeOffset` structures in C#. With .NET 8, there is now a built-in
`TimeProvider` for programs to use. Let's see how it works.

<!--more-->

### The Problem

I have a class that depends on knowing the current date/time from the system clock. The problem is
with testing the code, I cannot control the clock to simulate the scenarios I need to test.

Here is an example class:

```csharp
public class DataCache
{
    private readonly TimeSpan _cacheDuration;
    private readonly Func<string> _updateAction;
    private DateTimeOffset _lastUpdate;
    private string _cachedValue;

    public DataCache(TimeSpan cacheDuration, Func<string> updateAction)
    {
        _cacheDuration = cacheDuration;
        _lastUpdate = DateTimeOffset.MinValue;
        _cachedValue = string.Empty;
        _updateAction = updateAction;
    }

    public string GetCachedValue()
    {
        if (DateTimeOffset.UtcNow - _lastUpdate > _cacheDuration)
        {
            UpdateCache();
        }
        return _cachedValue;
    }

    private void UpdateCache()
    {
        _cachedValue = _updateAction();

        _lastUpdate = DateTimeOffset.UtcNow;
    }
}
```

It will update the cached value after the cache duration has been elapsed:

```csharp
var updateAction = new Func<string>(() => DateTime.UtcNow.ToString());

var cacheDuration = TimeSpan.FromSeconds(5);
var dataCache = new DataCache(cacheDuration, updateAction);

for (int i = 0; i < 10; i++)
{
    Console.WriteLine("Cached value: " + dataCache.GetCachedValue());
    Thread.Sleep(1000);
}
```

This will produce something like this:

```text
Cached value: 5/11/2025 9:22:32 AM
Cached value: 5/11/2025 9:22:32 AM
Cached value: 5/11/2025 9:22:32 AM
Cached value: 5/11/2025 9:22:32 AM
Cached value: 5/11/2025 9:22:32 AM
Cached value: 5/11/2025 9:22:37 AM
Cached value: 5/11/2025 9:22:37 AM
Cached value: 5/11/2025 9:22:37 AM
Cached value: 5/11/2025 9:22:37 AM
Cached value: 5/11/2025 9:22:37 AM
```

What does a unit test look like for this class?

```csharp
[Fact]
public void Cached_Value_Changes_After_Cached_Duration_Elapses()
{
    // Arrange
    var updateAction = new Func<string>(() => DateTimeOffset.UtcNow.ToString());

    var cacheDuration = TimeSpan.FromMicroseconds(5);
    var dataCache = new DataCache(cacheDuration, updateAction);

    // Act
    var firstValue = dataCache.GetCachedValue();
    Thread.Sleep(1500);
    var secondValue = dataCache.GetCachedValue();

    // Assert
    Assert.NotEqual(firstValue, secondValue);
}
```

In order to test this class, I need to include a `Thread.Sleep` (or `Task.Delay`) call in order to allow
the cache duration to expire. It doesn't matter how small the cacheDuration is. Granted, I could
change the `updateAction` to provide a unique value each time. But let's just assume that is not
something I am able or willing to change.

### Use Abstract Infrastructure Dependencies

The main problem with this class is its dependency on the `DateTimeOffset` structure. We have no way
to control its values. This makes unit tests difficult to construct or causes you to change the design
of your code to accommodate this dependency. In the past I would introduce an abstraction to remove
this dependency:

```csharp
public interface ISystemClock
{
    DateTimeOffset UtcNow
}
```

This would be injected into the class to remove the dependency:

```csharp
public class DataCache
{
    private readonly TimeSpan _cacheDuration;
    private readonly Func<string> _updateAction;
    private readonly ISystemClock _systemClock;
    private DateTimeOffset _lastUpdate;
    private string _cachedValue;

    public DataCache(TimeSpan cacheDuration, Func<string> updateAction, ISystemClock systemClock)
    {
        _cacheDuration = cacheDuration;
        _updateAction = updateAction;
        _systemClock = systemClock;
        _lastUpdate = DateTimeOffset.MinValue;
        _cachedValue = string.Empty;
    }

    public string GetCachedValue()
    {
        if (_systemClock.UtcNow - _lastUpdate > _cacheDuration)
        {
            UpdateCache();
        }
        return _cachedValue;
    }

    private void UpdateCache()
    {
        _cachedValue = _updateAction();

        _lastUpdate = _systemClock.UtcNow;
    }
}
```

Now I can simulate the passage of time using a mock system clock:

```csharp
[Fact]
public void Cached_Value_Changes_After_Cached_Duration_Elapses()
{
    // Arrange
    var mockClock = new Mock<ISystemClock>();

    // return different values for each call
    mockClock.SetupSequence(m => m.UtcNow)
        .Returns(DateTimeOffset.UtcNow)
        .Returns(DateTimeOffset.UtcNow.AddSeconds(1))
        .Returns(DateTimeOffset.UtcNow.AddSeconds(2))
        .Returns(DateTimeOffset.UtcNow.AddSeconds(3));

    var updateAction = new Func<string>(() => mockClock.Object.UtcNow.ToString());

    var cacheDuration = TimeSpan.FromMicroseconds(5);
    var dataCache = new DataCache(cacheDuration, updateAction, mockClock.Object);

    // Act
    var firstValue = dataCache.GetCachedValue();
    var secondValue = dataCache.GetCachedValue();

    // Assert
    Assert.NotEqual(firstValue, secondValue);
}
```

But now .NET provides another alternative, the `TimeProvider` class.

### Introducing the TimeProvider class

(Finally?) the .NET base libraries include a time abstraction we can use in our code. The `TimeProvider`
allows you to override the default implementation, making it possible to unit test the code simpler.

It provides a `GetUtcNow()` method which is a wrapper around the `DateTimeOffset.UtcNow` value.

We can swap out our `ISystemClock` abstraction for the `TimeProvider` and our class works the same:

```csharp
public class DataCache
{
    private readonly TimeSpan _cacheDuration;
    private readonly Func<string> _updateAction;
    private readonly TimeProvider _timeProvider;
    private DateTimeOffset _lastUpdate;
    private string _cachedValue;

    public DataCache(TimeSpan cacheDuration, Func<string> updateAction, TimeProvider timeProvider)
    {
        _cacheDuration = cacheDuration;
        _updateAction = updateAction;
        _timeProvider = timeProvider;
        _lastUpdate = DateTimeOffset.MinValue;
        _cachedValue = string.Empty;
    }

    public string GetCachedValue()
    {
        if (_timeProvider.GetUtcNow() - _lastUpdate > _cacheDuration)
        {
            UpdateCache();
        }
        return _cachedValue;
    }

    private void UpdateCache()
    {
        _cachedValue = _updateAction();

        _lastUpdate = _timeProvider.GetUtcNow();
    }
}
```

### Writing Tests with the TimeProvider

There are a few ways to use the `TimeProvider` in your tests. First, you can create a fake instance
with overloads on the methods required for your tests.

For example, in my case I want to control the values provided each time the `GetUtcNow()` method is
called. Here is a simple example:

```csharp
public class FakeTimeProvider : TimeProvider
{
    private readonly Stack<DateTimeOffset> _timeStack;
    private DateTimeOffset _currentTime;

    public FakeTimeProvider(DateTimeOffset[] values)
    {
        _timeStack = new Stack<DateTimeOffset>(values.Reverse());
    }

    public override DateTimeOffset GetUtcNow()
    {
        if (_timeStack.Count > 0)
        {
            _currentTime = _timeStack.Pop();
        }
        return _currentTime;
    }
}
```

Now the test can be written like this:

```csharp
[Fact]
public void Cached_Value_Changes_After_Cached_Duration_Elapses()
{
    // Arrange
    var fakeTimeProvider = new FakeTimeProvider(
    [
        DateTimeOffset.UtcNow,
        DateTimeOffset.UtcNow.AddSeconds(1),
        DateTimeOffset.UtcNow.AddSeconds(2),
        DateTimeOffset.UtcNow.AddSeconds(3)
    ]);

    var updateAction = new Func<string>(() => fakeTimeProvider.GetUtcNow().ToString());

    var cacheDuration = TimeSpan.FromMicroseconds(5);
    var dataCache = new DataCache(cacheDuration, updateAction, fakeTimeProvider);

    // Act
    var firstValue = dataCache.GetCachedValue();
    var secondValue = dataCache.GetCachedValue();

    // Assert
    Assert.NotEqual(firstValue, secondValue);
}
```

The other way to test with the `TimeProvider` is to use a fake version the .NET provides via a NuGet
package:

```powershell
dotnet add package Microsoft.Extensions.TimeProvider.Testing
```

The test code now looks like this:

```csharp
using Microsoft.Extensions.Time.Testing;

[Fact]
public void Cached_Value_Changes_After_Cached_Duration_Elapses()
{
    // Arrange
    var fakeTimeProvider = new FakeTimeProvider
    {
        AutoAdvanceAmount = TimeSpan.FromSeconds(1)
    };

    var updateAction = new Func<string>(() => fakeTimeProvider.GetUtcNow().ToString());

    var cacheDuration = TimeSpan.FromMicroseconds(5);
    var dataCache = new DataCache(cacheDuration, updateAction, fakeTimeProvider);

    // Act
    var firstValue = dataCache.GetCachedValue();
    var secondValue = dataCache.GetCachedValue();

    // Assert
    Assert.NotEqual(firstValue, secondValue);
}
```

The `FakeTimeProvider` in the NuGet package covers most test scenarios you could need to simulate. I
would recommend using this provider in your tests rather than creating your own custom versions. But
I understand if you don't want to take on another dependency to your test project if you already are
using a mocking library like `Moq`. For example, this is also possible to do:

```csharp
var mockClock = new Mock<FakeTimeProvider>();

// return different values for each call
mockClock.SetupSequence(m => m.GetUtcNow())
    .Returns(DateTimeOffset.UtcNow)
    .Returns(DateTimeOffset.UtcNow.AddSeconds(1))
    .Returns(DateTimeOffset.UtcNow.AddSeconds(2))
    .Returns(DateTimeOffset.UtcNow.AddSeconds(3));

var fakeTimeProvider = mockClock.Object;
```

### Summary

As a general rule, your code should rely on abstractions for external dependencies, like the system
clock. The new `TimeProvider` class included in .NET provides such an abstraction and it makes
testing time-based code straightforward. I am glad I don't have to create my own abstractions
any longer.
