---
layout: post
title: 'Customizing OpenAPI Documentation In ASP.NET - Part 1'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

The default OpenAPI documentation generated from your ASP.NET code is rarely correct. Or rather, it
is rarely what you want it to be. Let's see how we can make it better.

<!--more-->

### I Only speak JSON

By default, the generated OpenAPI document will think your API is capable of responding in many different
forms:

- text/plain
- text/json
- application/json

And it assumes you will accept these types of payloads:

- application/json
- text/json
- application/*+json

If you only want to accept and respond with `application/json` then you can decorate your controllers
with the following:

```csharp
// using System.Net.Mime;
[Produces(MediaTypeNames.Application.Json)]
[Consumes(MediaTypeNames.Application.Json)]
```

### Endpoints Do More Than It Thinks

For a `GET` endpoint, the generated OpenAPI document assumes a response code of 200 OK. The same goes
for a `POST` endpoint. But what if you return a 404 NOT FOUND, or 400 BAD REQUEST, or 201 CREATED?

We can (and should) tell the code generator what status codes each endpoint can respond with. Decorate
each controller action method with the correct response types:

```csharp
[HttpGet("{id}")]
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]

[HttpPost]
[ProducesResponseType(StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
```

What about global response type, like 401 UNAUTHORIZED, 403 FORBIDDEN, and 500 INTERNAL SERVER ERROR?

You can set them globally rather than each controller/action when adding support for controllers:

```csharp
// using Microsoft.AspNetCore.Mvc;
builder.Services.AddControllers(configure =>
{
    configure.Filters.Add(
       new ProducesResponseTypeAttribute(StatusCodes.Status500InternalServerError));
    configure.Filters.Add(
       new ProducesResponseTypeAttribute(StatusCodes.Status401Unauthorized));
    configure.Filters.Add(
       new ProducesResponseTypeAttribute(StatusCodes.Status403Forbidden));
});
```

### We All Have Problems

Recently I've done a lot of experimentation with using Problem Details (see my previous 2 posts). If
you have configured all 400 and 500 level response to include the Problem Details response payloads,
you must also include those details in the generated OpenAPI document.

This can be accomplished by changing the global controller filters as follows:

```csharp
    configure.Filters.Add(
       new ProducesResponseTypeAttribute(
            typeof(ProblemDetails),
            StatusCodes.Status500InternalServerError,
            MediaTypeNames.Application.ProblemJson));
    configure.Filters.Add(
       new ProducesResponseTypeAttribute(
            typeof(ProblemDetails),
            StatusCodes.Status401Unauthorized,
            MediaTypeNames.Application.ProblemJson));
    configure.Filters.Add(
       new ProducesResponseTypeAttribute(
            typeof(ProblemDetails),
            StatusCodes.Status403Forbidden,
            MediaTypeNames.Application.ProblemJson));
```

The problem (no pun intended) with this approach is it's in conflict with using the `Produces` attribute:

```csharp
[Produces(MediaTypeNames.Application.Json)]
```

If you use this attribute, it forces ALL responses to be described as `application/json` in the
generated OpenAPI document. This may or may not be acceptable in your context. If you want to ensure
that the 400/500 level responses are documented with a content of `application/problem+json`, then you
cannot use the `Produces` attribute. Instead you must define the content type for each status code
by updating the `ProducesResponseType` attribute to be more explicit:

```csharp
// [ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(typeof(WeatherForecast), StatusCodes.Status200OK, MediaTypeNames.Application.Json)]
```

### What Did I Miss?

There is a built-in code analyzer you can enable to help you find OpenAPI attributes you may have
missed. In your CSPROJ file, add the following:

```xml
<PropertyGroup>
 <IncludeOpenAPIAnalyzers>true</IncludeOpenAPIAnalyzers>
</PropertyGroup>
```

You will now see (at build-time) warnings for any missing OpenAPI attributes. For example you will
receive a warning if your API uses the `NotFound` method to return a 404 response but there is no
`ProducesResponseType` attribute with the 404 status code. It isn't foolproof, but it might help you
spot some obvious omissions.

### Summary

These are just a handful of the possible customizations you can make to the OpenAPI document generated
from the code. They help align the generated document to the actual code which helps minimize any
confusion or unexpected behavior.
