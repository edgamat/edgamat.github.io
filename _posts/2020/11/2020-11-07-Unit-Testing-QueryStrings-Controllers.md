---
layout: post
title: 'Unit Testing in Controllers Part 1: QueryString'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
The ASPNET Core approach to processing HTTP requests is to abstract as much as possible. As a result, it is rare that you have to use anything but POCO classes when processing a request with a controller action. All the HTTP-ness is nicely hidden behind model binders and value providers that parses the incoming request (query string, form, body, url, etc.). This makes it possible to unit test your controllers without having to create mock implementations of the HTTP request itself (ie. the HttpContext).

But this doesn't work in all cases. 

<!--more-->

### The Problem

I had the need to obtain the raw Query String from the HTTP request in a controller action. So in the action method, I used the HttpContext to obtain the value:

```csharp
var queryString = HttpContext.Request.QueryString.ToString();
```

This worked perfectly. However, the unit test for the method now failed because the `HttpContext` controller property was `null`. Some quick searches on the web resulted in a simple looking solution. The advice was to add an HttpContext to my controller under test:

```csharp
controller.ControllerContext = new ControllerContext
{
    HttpContext = new DefaultHttpContext()
}
```

And this solved part of my problem. The unit test didn't throw an exception any longer. However, the value of `queryString` was always an empty string. So I needed to find a way to set the value so my test would pass.

### The Solution

You can set the query string value by explicitly creating the `QueryString` property. There is a constructor that accepts a dictionary of query string parameters and results in a populated (ie. non-empty) `QueryString`.

```csharp
var context = new DefaultHttpContext();

var parameters = new Dictionary<string, StringValues>
{
    { "id", "12345" },
    { "token", "QWERTY" }
};

context.Request.QueryString = QueryString.Create(parameters);

controller.ControllerContext = new ControllerContext
{
    HttpContext = context
}
```

Now the unit tests pass because the code correctly simulates a non-empty query string for the incoming request.

### Side Note

This approach also simulates the Query parameter of the request:

```csharp
var queryStringToken = HttpContext.Request.Query["token"];
```

However, I found a lot of documentation online and examples showing a different approach, using the `IQueryFeature`:

```csharp
var parameters = new Dictionary<string, StringValues>
{
    { "id", "12345" },
    { "token", "QWERTY" }
};

var queryCollection = new QueryCollection(parameters);
var query = new QueryFeature(queryCollection);

var features = new FeatureCollection();
features.Set<IQueryFeature>(query);
features.Set<IHttpRequestFeature>(new HttpRequestFeature());

var context = new DefaultHttpContext(features);
```

Unfortunately, this approach only simulated the `Query` property behavior, and not the `QueryString` property. In order to simulate the `QueryString` it is necessary to take an additional step and set the RequestFeature property:

```csharp
var parameters = new Dictionary<string, StringValues>
{
    { "id", "12345" },
    { "token", "QWERTY" }
};

var queryCollection = new QueryCollection(parameters);
var query = new QueryFeature(queryCollection);

var request = new HttpRequestFeature
{
    QueryString = "?id=12345&token=QWERTY"
};

var features = new FeatureCollection();
features.Set<IQueryFeature>(query);
features.Set<IHttpRequestFeature>(request);

var context = new DefaultHttpContext(features);
```

I prefer the first method I described because it produces the same result and I don't have to manually encode the query string parameters into a string. But I could see cases where either approach could be preferred.

### Summary

Hopefully you don't need to simulate HttpContext behavior in controller test cases very often. But if you do, it is nice to know that there are ways to simulate the requests properly.

This was Part 1. In Part 2 I'll look at simulating the `User` property of a controller.
