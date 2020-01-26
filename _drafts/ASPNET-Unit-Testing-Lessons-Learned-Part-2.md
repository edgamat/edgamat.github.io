---
layout: post
title: 'ASPNET Core Unit Testing Lessons Learned Part 2'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When building a Web API endpoint with ASPNET Core, it is desirable to keep the Controller classes
as small as possible. The important logic should be in business classes or input/view models. What 
remains in the Controllers should be simple. But, they still require unit tests in some situations. 
Let's look at some of these patterns.

<!--more-->

## Controller Actions

A `GET` action on a controller class will typically have the following pattern:

```csharp
public async Task<IActionResult> GetUser(string userId, CancellationToken token)
{
    var user = await _userService.GetByUserIdAsync(userId, token);
    if (user == null)
    {
        return NotFound("invalid-user");
    }

    var userResponseModel = user.ToViewModel();

    return Ok(userResponseModel);
}
```

What 'deserves' a unit test for this method? Tests could be written to cover all the return types. Tests
could be written to inspect the contents of the response. 

Each team must use its own judgement to decide what needs a unit test. As long as `GetByUserIdAsync` and `ToViewModel`
have good coverage with unit tests, it may not be necessary to explicitly test this `GET` method.

That is why the design of controller actions has such a huge impact on whether they requires unit tests. If you can
remove all but the essential controller responsibilities, most action methods become so simple that unit tests
are not going to be necessary.

## Faking What The Test Needs

Using the .NET Framework versions of MVC, controller tests could be challenging to write because creating a test
 subject (i.e. an instance of a controller) required the creation of a fake HTTP context for the class to 'work'.

.NET Core makes most of that difficulty go away, but doesn't eliminate it completely. It depends if the
action code makes use of the 'base' controller features. Here is an example where the action uses
functionality of the `ControllerBase` class:

```csharp
public async Task<IActionResult> UpdateAsync(string userId, [FromBody] UserRequestModel model, CancellationToken token)
{
    TryValidateModel(model);
    if (!ModelState.IsValid)
        return BadRequest(ModelState);
        
    var user = await _userService.GetByUserIdAsync(userId, token);
    if (user == null)
        return NotFound("invalid-user");

    if (!user.IsActive)
        ModelState.AddModelError(null, "Cannot update inactive users");

    if (!ModelState.IsValid)
        return BadRequest(ModelState);

    var updateUserCommand = model.ToUpdateUserCommand();

    await _userService.UpdateAsync(updateUserCommand, token);

    return Ok();
}
``` 

**NOTE** The default behavior for ASPNET Core MVC controllers is to automatically validate incoming models before
executing the action. However, in this example, that feature has been turned off in order to apply custom validation:

```csharp
services.Configure<ApiBehaviorOptions>(options =>
{
    options.SuppressModelStateInvalidFilter = true;
});
```

In this example, the action uses the `ModelState` and `TryValidateModel` members of the `ControllerBase` class. `ModelState` is available, without creating a test double, however, the `TryValidateModel` is not. If you attempt to call this function from a unit test, you'll get a runtime exception:

```
System.NullReferenceException : Object reference not set to an instance of an object.
   at Microsoft.AspNetCore.Mvc.ControllerBase.TryValidateModel(Object model, String prefix)
```

The `TryValidateModel` requires an implementation of an `IObjectModelValidator` set to the `ControllerBase.ObjectValidator` member.

When creating the system under test (ie. the controller) you can use a fake validator to make the test work (using the `Moq`
mocking framework):

```csharp
public static IObjectModelValidator CreateFakeObjectValidator(ControllerBase controller)
{
    var objectValidator = new Mock<IObjectModelValidator>();
    objectValidator.Setup(o => o.Validate(It.IsAny<ActionContext>(),
        It.IsAny<ValidationStateDictionary>(),
        It.IsAny<string>(),
        It.IsAny<object>()))
    .Callback((ActionContext actionContext, ValidationStateDictionary validationState, string prefix, object model) =>
    {
        var validationResults = new List<ValidationResult>();
        var valid = Validator.TryValidateObject(model, new ValidationContext(model), validationResults, true);

        foreach (var result in validationResults)
            controller?.ModelState.AddModelError(result.MemberNames.FirstOrDefault(), result.ErrorMessage);
    });

    return objectValidator.Object;
}

//  Act (in test case)
var sut = new MyController();
sut.ObjectValidator = MoqHelpers.CreateFakeObjectValidator(sut);

var actionResult = sut.UpdateAsync("1234", new UserRequestModel(), CancellationToken.None)
```

## Be Explicit

Assertions on action results need to be as explicit as possible. If not, you leave the test cases open to
returning a false positive (false failed tests), or worse a false negative (missed bugs that don't fail a test).

For example, in the `UpdateAsync` method, there are two paths through the code that result in a BadRequest ActionResult
returned from the method. If an assertion only checks for the proper ActionResult type, a bug could be present in the
code:

```csharp
//Act
var actionResult = sut.UpdateAsync("1234", new UserRequestModel(), CancellationToken.None)

//Assert
Assert.IsType<BadRequestObjectResult>(actionResult);
```

A more precise test would be to confirm the contents of the result. The test case can then look at
 the results are decide if the errors returned from the action are as expected:

```csharp
//Assert
var result = Assert.IsType<BadRequestObjectResult>(actionResult);

var badRequestData = (SerializableError)result.Value;
Assert.Equal("expected-error", ((string[])badRequestData["MyProperty"])[0]);
```

**NOTE** In some cases the `BadRequestObjectResult` is returning an anonymous object.
In order to inspect these objects, it is necessary to allow the unit testing assemble access to the internal types 
of the assembly where the controllers are declared. I usually add an `AssemblyInfo.cs` file to contain such declarations:

```csharp
[assembly: InternalsVisibleTo("SampleProject.UnitTests")]
```

For this anonymous object:

```csharp
var errors new string[] { "expected-error" };
return BadRequest(new { TheErrors = errors })
```

Here is the test code:

```csharp
//Assert
var result = Assert.IsType<BadRequestObjectResult>(actionResult);

dynamic badRequestData = result.Value;
Assert.Equal("expected-error", badRequestData.TheErrors[0]);
```

## Issues and Observations

As a lesson learned, I recommend spending time refactoring a controller method and 
remove all but the essential responsibilities. This makes testing the methods mush easier
or sometimes not necessary at all.

When testing controllers, be as explicit as possible. Ensure to assert the contents of the results
are correct where possible. This guards against false negatives which can keep bugs
hidden from tests.

