---
layout: post
title: 'Object Level Authorization in ASP.NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Broken Object Level Authorization is a huge security problem. Let's explore the options available
to us in an ASP.NET application

<!--more-->

### Object Level Authorization

According to [owasp.org](https://owasp.org), "Object level authorization is an access control
mechanism that is usually implemented at the code level to validate that a user can only
access the objects that they should have permissions to access." In other words, let's say you create
a new object using a POST endpoint. Object level authorization ensures that only you can retrieve
the details of that object (e.g. via a GET endpoint). If someone else attempts to retrieve the
object, the system should forbid them from doming so.

Let's create a simple example with the following controller methods:

```csharp
[HttpPost]
public IActionResult CreateWidget([FromBody] Widget widget)
{
    var userId = GetUserId(User);
    if (userId == Guid.Empty)
    {
        return BadRequest();
    }

    widget.OwnerId = userId;

    _widgetService.CreateWidget(widget);

    return CreatedAtAction(nameof(GetWidget), new { id = widget.Id }, widget);
}

[HttpGet("{id}")]
public IActionResult GetWidget(int id)
{
    var widget = _widgetService.GetWidget(id);
    return widget is null
        ? NotFound()
        : Ok(widget);
}
```

GetUserId is an extension method used to retrieve the UserId from the security principal derived
from the incoming HTTP request:

```csharp
public static Guid GetUserId(this ClaimsPrincipal principal)
{
    var subject = principal.FindFirstValue(JwtRegisteredClaimNames.Sub);

    return Guid.TryParse(subject, out var userId)
        ? userId
        : Guid.Empty;
}
```

Notice that the `GetWidget` method doesn't check that the UserId of the User equals the OwnerId
of the object. This is what object level authorization protects against. Without a check to confirm
ownership of the object, the API could return widgets created by other users, which is a serious
security problem.

### The simplest options

The first approach to solving this problem is the obvious one; if the UserId and OwnerId are different
then don't return the object:

```csharp
[HttpGet("{id}")]
public IActionResult GetWidget(int id)
{
    var widget = _widgetService.GetWidget(id);
    if (widget is null)
    {
        return this.NotFound();
    }

    var userId = this.GetUserId(this.User);

    // confirm object level authorization
    if (widget.OwnerId != userId)
    {
        return this.NotFound();
    }

    return this.Ok(widget);
}
```

Another simple option is to modify the Widget Service and require the caller to provide the OwnerId
when querying the database for the widgets:

```csharp
[HttpGet("{id}")]
public IActionResult GetWidget(int id)
{
    var ownerId = this.GetUserId(User);

    var widget = _widgetService.GetWidget(ownerId, id);
    return widget is null
        ? this.NotFound()
        : this.Ok(widget);
}
```

Both of these options will provide the necessary protections to the objects. Only the owner of the
object can retrieve the object. However, they both have drawbacks. The first option makes the
controller responsible for making security decisions, while the second option delegates that
responsibility to the Widget Service. Controllers should not contain security rules, as that is
not their primary function. And similarly, the Widget Service should only be responsible for
orchestrating the persistence of the objects to/from the database. If we place security rules
in the controllers and service classes, we will end up with rules scattered throughout the code.
This adds additional complexity to the code, as you need to account for these embedded rules in your
unit test cases. It also makes it more difficult to find and understand the security rules, when
changes need to be made.

### Centralized Authorization using Policies

The ASP.NET Core libraries include a framework for authorizing requests. It is called
"Resource-based authorization":

[https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased)

Authorization decisions are made via a separate service that can be accessed by injecting a
`IAuthorizationService` instance into the controller:

```csharp
public class WidgetController : ControllerBase
{
    private readonly IAuthorizationService _authorizationService;
    private readonly IWidgetService _widgetService;

    public DocumentController(IAuthorizationService authorizationService,
                              IWidgetService widgetService)
    {
        _authorizationService = authorizationService;
        _widgetService = widgetService;
    }
    ...
}
```

The `GetWidget` action method can use this service to determine if the user has access to the requested
resource:

```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetWidget(int id)
{
    var widget = _widgetService.GetWidget(id);
    if (widget is null)
    {
        return this.NotFound();
    }

    var authorizationResult = await _authorizationService
            .AuthorizeAsync(User, widget, "SameOwnerPolicy");

    // confirm object level authorization
    if (!authorizationResult.Succeeded)
    {
        return this.NotFound();
    }

    return this.Ok(widget);
}
```

Introducing this new abstraction relocates where the authorization rules are located in the code.
You no longer embed the rules in the controller, nor in the service classes. The rules can be located
in a central location in the code, making reasoning about the rules and changing them easier.

### Policies, Requirements and Handlers

The authorization logic that was in the controller/service is now located in a new class, called a
authorization handler. It is executed by the implementation of the IAuthorizationService and
determines if authorization should be granted.

In our case, we want to make an handler that checks that the owner of the Widget matches the UserId
from the incoming HTTP request.

We start by defining an authorization requirement. It is a marker class that is used to register
the handler with the policy:

```csharp
using Microsoft.AspNetCore.Authorization;

public class SameOwnerRequirement : IAuthorizationRequirement
{
}
```

Now, we code the handler:

```csharp
using Microsoft.AspNetCore.Authorization;

public class WidgetAuthorizationHandler :
    AuthorizationHandler<SameOwnerRequirement, Widget>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
                                                   SameOwnerRequirement requirement,
                                                   Widget resource)
    {
        if (context.User.GetUserId() == resource.OwnerId)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

Note, a separate handler is needed for each type of resource (Widget, Sprocket, etc.).

And now register the handler with the policy in the `Program.cs`:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("SameOwnerPolicy", policy =>
        policy.Requirements.Add(new SameOwnerRequirement()));
});

builder.Services.AddSingleton<IAuthorizationHandler, WidgetAuthorizationHandler>();
```

This is a lot of extra code, compared to the original 'simplest options' first introduced. Why is all
this necessary? Well, I would offer that this isn't really a lot of additional code. If moves the
authorization logic to a central place (say an "Authorizations" folder in your Web API project). All
the authorization rules can be located here. If you need to understand the rules, you can find them
all here. The controller can now easily be tested because the input test data doesn't need to include
the necessary claims to ensure the authorization rules are in place. And if you change the authorization
rules, that additional complexity is not the responsibility of the controller, keeping them more
understandable and maintainable.

### Summary

Providing object level authentication is possible through many different options in ASP.NET. The
built-in authorization feature, like policies, handlers and requirements keep the rules separate
from the rest of the code which makes the code easier to test, understand and change. Give them a
try!
