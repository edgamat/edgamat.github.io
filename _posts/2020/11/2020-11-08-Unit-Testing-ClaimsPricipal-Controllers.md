---
layout: post
title: 'Unit Testing in Controllers Part 2: ClaimsPrincipal'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
In the first part of this mini-series, I showed how I created a simulated HttpContext for a controller action unit test. In this post I wanted to describe how I added a ClaimsPrincipal implementation.

<!--more-->

### The Problem

I had the need to obtain the identity of the authenticated user for the current request in a controller action. So in the action method, I used the built-in `User` to obtain the value:

```csharp
var userName = User.Identity.Name;
```

This worked perfectly. However, as with the QueryString property I simulated in Part 1 of this series, the unit test for the method now failed because the `User.Identity.Name` property was `null`.

### The Solution

You can set the `User` value by explicitly creating the `ClaimsPrincipal` and assigning it to the HttpContext.

```csharp
var context = new DefaultHttpContext();

var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, "username"),
};
var identity = new ClaimsIdentity(claims, "TestAuthType");
var claimsPrincipal = new ClaimsPrincipal(identity);

context.User = claimsPrincipal;

controller.ControllerContext = new ControllerContext
{
    HttpContext = context
}
```

Now the unit tests pass because the code correctly simulates a non-empty identity name for the incoming request.

### Summary

Simulating portions of the HttpContext for controller actions should hopefully be few and far between. But it is nice to know it is possible to do it when necessary.
