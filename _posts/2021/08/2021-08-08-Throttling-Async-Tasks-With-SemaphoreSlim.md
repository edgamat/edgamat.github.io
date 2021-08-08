---
layout: post
title: 'Throttling Async Tasks with SemaphoreSlim'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Recently we had a situation where we thought we were throttling tasks correctly. Turns out we weren't.

<!--more-->

One of the microservices we support receives messages from a message broker. We configured the consumer to create a maximum of 8 receiving clients. We assumed that this would limit the number of messages concurrently being consumed at 8. The reason this was important is because each message had to call a third-party API via HTTP and the maintainers of this API had asked us to limit the number of concurrent calls. 

But once this solution was deployed, the third-party API maintainers indicated they saw over 150 concurrent requests being sent from our application. At first we were very confused. How did our 8 clients produce 150 concurrent requests? 

The NuGet package we are using to consume messages is where we found the cause. We had configured it to use 8 clients. But what we didn't know is that each client pulls in a batch of messages at a time and process each message in parallel using async/await tasks. So our 8 clients were processing dozens of messages at the same time, resulting in the higher than expected concurrent requests being sent to the third-party API.

We needed an additional control mechanism to limit the number of requests.

### What is a Semaphore?

In the .NET world, a semaphore limits the number of threads that can access a resource concurrently. It is a means of preventing race conditions or ensuring thread safety when accessing a shared set of resources. 

The semaphore keeps track of how many units of a resource are available. The code waits for the semaphore to let it know when unit of a resource is available. Once a unit is available, the semaphore reduces the number of available units and the code executes. When the code is finished, it lets the semaphore know it no longer needs the resource. The semaphore releases the unit which increases the number of available units.

This is all done in a thread-safe way which enables the code to share a limited number of resources across multiple threads.

Let's say that you have a cache. You want to ensure that only one thread can update the cache at a time (thus making it thread-safe). A semaphore can help you accomplish this:

```csharp
private readonly SemaphoreSlim Lock = new SemaphoreSlim(1, 1);

private readonly IDictionary<string, Session> Cache = new Dictionary<string, Session>();

public async Task<Session> GetSessionAsync(string sessionId, CancellationToken token)
{
    if (Cache.ContainsKey(sessionId))
    {
        return Cache[sessionId];
    }
    
    await Lock.WaitAsync(token);

    try
    {
        var session = await CreateSessionAsync(sessionId, token);
        
        Cache.Add(sessionId, session);
    }
    finally
    {
        Lock.Release();
    }
    
    return Cache[sessionId];
}
```

In this case, the semaphore is allowing a maximum of 1 thread to create a session (ie. update the cache).

It is important that each 'wait' has a corresponding 'release':

```csharp
await Lock.WaitAsync(token);
...
...
Lock.Release();
```

### Throttling HTTP Requests

Back to our situation, we wanted to limit the number of concurrent HTTP requests the code could make. We introduced a semaphore to achieve that:

```csharp
private readonly SemaphoreSlim Throttle = new SemaphoreSlim(8, 8); 
```

And added the wait/release calls to the tasks used to call the third-party API:

```csharp
await Throttle.WaitAsync(token);

try
{
    // Send HTTP Request
}
finally
{
    Throttle.Release();
}
```

The semaphore ensured that no more than 8 HTTP requests could be made concurrently. And this successfully throttled the requests so we didn't overload the resources of the third-party API.

There were other places where throttling was needed so we added a semaphore in these cases as well. In addition, we added configuration parameters to control the size of each semaphore so the throttling could be reconfigured as needed. 
