---
layout: post
title: 'Using Application State in ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

"Classic" ASP.NET applications would use the built-in `Application` object to store values that were cached across the entire application. In ASP.NET Core, this was removed. Let's look at alternatives.

<!--more-->

### The Problem

Recently I had a need to store state in an ASP.NET Core HTTP Web API application. I required incoming requests to check the state of the data store. If the data was being refreshed, which occurred once every couple of weeks, the application needed to respond with a 'service unavailable' response. A background hosted service (`IHostedService`) was responsible for refreshing the data, triggered by a message it would receive to begin the refresh work.

The approach I decided on was a _Singleton_ service that both the background service (writer) and controllers (readers) could share a common set of state variables:

```csharp
public interface IApplicationState
{
    public TEntry Get<TEntry>(string key);

    public void Set<TEntry>(string key, TEntry entry);
}
```

The background service would be responsible for setting the state:

```csharp
if (_messages.TryDequeue(out message))
{
    _applicationState.Set<bool>("IsRefreshing", true);
    
    try
    {
        await ProcessMessageAsync(message, token);
    }
    catch (Exception ex)
    {
        _log.LogError(ex, "Exception processing message {id}", message.Id)
    }
    finally
    {
        _applicationState.Set<bool>("IsRefreshing", false);
    }
}
```

In this context, only one instance of the background service is running at a time. Within that service, only one message is being processed at a time. This simplifies a lot of the threading/concurrency issues. 

The messages occur infrequently, so the controller actions will normally respond with the results of accessing the external service

The controller actions use the application state to response correctly to incoming requests:

```csharp
if (_applicationState.Get<bool>("IsRefreshing"))
{
    return StatusCode(StatusCodes.Status503ServiceUnavailable, new
    {
        Message = "data_being_refreshed"
    });
}
```

### MemoryCache as a Solution

The `MemoryCache` is part of the memory caching library provided by .NET Core. It is thread safe and has several extension methods that make it well suited for the type of cache I require.

This includes methods to configure the memory cache using the Dependency Injection framework:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMemoryCache();
}
```

This provides an implementation of the `IMemoryCache` using the typical constructor injection pattern:

```csharp
public class MemoryCacheApplicationState : IApplicationState
{
    private readonly IMemoryCache _cache;

    public ApplicationState(IMemoryCache cache)
    {
        _cache = cache;
    }

    public TEntry Get<TEntry>(string key)
    {
        return _cache.TryGetValue(key, out TEntry entry)
            ? entry
            : default;
    }

    public TEntry Set<TEntry>(string key, TEntry entry)
    {
        return _cache.Set(key, entry);
    }
}
```

### Context

Does this solution work as a universal application cache? I honestly don't know. I doubt it. But for this context, it is simple, and handles the task at hand. A cache that is written to by one thread and read from by multiple threads. It is possible to simplify it further by using a `ConcurrentDictionary<T>` class. But I think this solution is simple enough and can be read and understood easily. 
