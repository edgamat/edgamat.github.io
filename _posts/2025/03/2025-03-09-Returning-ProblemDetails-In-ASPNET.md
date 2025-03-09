---
layout: post
title: 'Returning ProblemDetails In ASP.NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Continuing my research from last week [Using ProblemDetails in ASP.NET]({% post_url /2025/03/25-03-02-Using-ProblemDetails-In-ASPNET %}),
I want to include ProblemDetails in other scenarios. For example, when a user receives a 401 or 403 response,
there is no payload included in the response. I would prefer to include a ProblemDetails payload,
making it simpler for callers to use the API.

<!--more-->

### Use Status Code Pages

To have ASP.NET return ProblemDetails for 401 and 403 responses, include the Status Code Pages in
the request pipeline:

```csharp
var app = builder.Build();

app.UseStatusCodePages();

app.UseExceptionHandler();
```

### More Details Please

When you are using the API locally, you may wish to have it return more details for the 500 Internal
Server Error responses. Again, it is very simple to do, using the Developer Exception Page:

```csharp
var app = builder.Build();

app.UseStatusCodePages();

if (app.Environment.IsEnvironment("Local"))
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler();
}
```

### Further Customizations

Both the Status Code Pages and Exception Handler allow you to customize the responses even further.
If you do not include the `AddProgramDetails()` when building your application, the `app.UseStatusCodePages()`
will use `text/plain` for the response payload. So you may want to change how it behaves.

For the Status Code Pages, you can add your customizations using a custom handler:

```csharp
app.UseStatusCodePages(handler =>
{
    var statusCode = handler.HttpContext.Response.StatusCode;
    var problemDetails = new ProblemDetails
    {
        Status = statusCode,
        Title = ReasonPhrases.GetReasonPhrase(statusCode),
        Detail = "A problem occurred while processing your request."
    };

    return Results.Problem(problemDetails).ExecuteAsync(handler.HttpContext);
});
```

But I would not recommend this, instead I would build the application with the `AddProgramDetails()`
included. By default, the call to `app.UseStatusCodePages()` uses the configured
`IProblemDetailsService` to produce the response payload. It will include all your ProblemDetails
customizations automatically.

And for the Exception Handler, you can customize it too:

```csharp
app.UseExceptionHandler(new ExceptionHandlerOptions
{
    ExceptionHandler = async context =>
    {
        var statusCode = context.Response.StatusCode;
        var problemDetails = new ProblemDetails
        {
            Status = statusCode,
            Title = ReasonPhrases.GetReasonPhrase(statusCode),
            Detail = "A problem occurred while processing your request"
        };
        await Results.Problem(problemDetails).ExecuteAsync(context);
    }
});
```

And similarly to the `app.UseStatusCodePages()`, the `app.UseExceptionHandler()` will, by default
use the the configured `IProblemDetailsService` to produce the response payload. If all you want is
to control the way ProblemDetails are rendered for exceptions, it is best to include those customizations
in the ProblemDetails configuration, and leave the `app.UseExceptionHandler()` with the default
settings.

### Model Binding with JSON Property Names

A controller that is decorated with the `ApiController` attribute will automatically validate the
request payload and will return a 400 Bad Request when it encounters an invalid payload:

```csharp
public class CreateWidgetRequest
{
    [Required]
    [JsonPropertyName("widgetName")]
    public string Name { get; init; } = null!;
}
```

By default, if a model validation error occurs, the response will not use the `JsonPropertyName`
value:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "detail": "Please provide the 'traceid' when contacting us.",
  "errors": {
    "Name": [
      "The Name field is required."
    ]
  },
  "traceId": "00-26908c4b78f7284d82643cfd8c1b75d8-5f0ff53c435fa2eb-00"
}
```

But you can change the configuration to use the `JsonPropertyName` attributes:

```csharp
using Microsoft.AspNetCore.Mvc.ModelBinding.Metadata;

...

builder.Services.AddControllers(configure =>
{
    configure.ModelMetadataDetailsProviders.Add(new SystemTextJsonValidationMetadataProvider());
});
```

Now the expected property name is used:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "detail": "Please provide the 'traceid' when contacting us.",
  "errors": {
    "widgetName": [
      "The widgetName field is required."
    ]
  },
  "traceId": "00-a00c672664dbb4d0531a6ca86d68b0d3-b73456d919873338-00"
}
```

### Manual Validation Errors

In action controllers you can return a `ValidationProblem` which will construct a ProblemDetails
payload for you. One way is to pass in a `ValidationProblemDetails` object:

```csharp
if (request.Name.Contains("a bad word"))
{
    var errors = new Dictionary<string, string[]>
    {
        { "widgetName", ["widgetName cannot contain a bad word"] }
    };

    var problemDetails = new ValidationProblemDetails(errors);
    return ValidationProblem(problemDetails);
}
```

Unfortunately, this overload of the `ValidationProblem` method does not use the configured
`IProblemDetailsService`. The result is inconsistent with the other ProblemDetails:

```json
{
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "widgetName": [
      "widgetName cannot contain a bad word"
    ]
  }
}
```

However, there is another overload, that use the `ModelStateDictionary` that does use the
configured `IProblemDetailsService`:

```csharp
if (request.Name.Contains("a bad word"))
{
    ModelState.AddModelError("widgetName", "widgetName cannot contain a bad word");

    return ValidationProblem(ModelState);
}
```

Now we receive the expected payload, with all our customizations to the ProblemDetails:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "detail": "Please provide the 'traceid' when contacting us.",
  "errors": {
    "widgetName": [
      "widgetName cannot contain a bad word"
    ]
  },
  "traceId": "00-2b37877913ecb5cd4abb892f84be2ec5-ed4488781e027c6b-00"
}
```

### Conflict Has To Be Different

When using the built in `Conflict` response methods, you would expect to get something similar to
the ValidationProblem response. But you'd be wrong. For example:

```csharp
if (_widgetService.GetWidgets().Any(x => x.Name.Equals(request.Name, StringComparison.OrdinalIgnoreCase)))
{
    ModelState.AddModelError("widgetName", "A different widget is already using this name.");

    return Conflict(ModelState);
}
```

I was not expecting this as the output:

```json
{
  "widgetName": [
    "A different widget is already using this name."
  ]
}
```

Well, maybe we can do wrap the `ModelState` in a `ValidationProblemDetails`?

```csharp
if (_widgetService.GetWidgets().Any(x => x.Name.Equals(request.Name, StringComparison.OrdinalIgnoreCase)))
{
    ModelState.AddModelError("widgetName", "A different widget is already using this name.");

    return Conflict(new ValidationProblemDetails(ModelState));
}
```

Better, but not exactly what we wanted:

```json
{
  "title": "One or more validation errors occurred.",
  "status": 409,
  "errors": {
    "widgetName": [
      "A different widget is already using this name."
    ]
  }
}
```

To use our customized ProblemDetails, we can reuse the `ValidationProblem` method:

```csharp
if (_widgetService.GetWidgets().Any(x => x.Name.Equals(request.Name, StringComparison.OrdinalIgnoreCase)))
{
    ModelState.AddModelError("widgetName", "A different widget is already using this name.");

    return ValidationProblem(statusCode: StatusCodes.Status409Conflict, modelStateDictionary: ModelState);
}
```

And now we get the desired result:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.10",
  "title": "One or more validation errors occurred.",
  "status": 409,
  "detail": "Please provide the 'traceid' when contacting us.",
  "errors": {
    "widgetName": [
      "A different widget is already using this name."
    ]
  },
  "traceId": "00-3210edae71164b971bde2c29afd247f3-6b04a93acdecfca6-00"
}
```

You can use other status codes with the `ValidationProblem` besides 409, like 422 Unprocessable Entity:

```csharp
if (_widgetService.GetWidgets().Any(x => x.Name.Equals(request.Name, StringComparison.OrdinalIgnoreCase)))
{
    ModelState.AddModelError("widgetName", "A different widget is already using this name.");

    return ValidationProblem(statusCode: StatusCodes.Status422UnprocessableEntity, modelStateDictionary: ModelState);
}
```

### Summary

Use the default `UseStatusCodePages` and `UseExceptionHandler` middleware to ensure your Web API
uses your customized ProblemDetails payloads for all 400-level and 500 Internal Server Error
responses:

```csharp
builder.Services.AddProblemDetails(options =>
    options.CustomizeProblemDetails = (context) =>
    {
      // All the customizations in one place
        ApplyCustomizations(context.ProblemDetails);
    });
...

var app = builder.Build();

// Use the default middleware
app.UseStatusCodePages();

if (app.Environment.IsEnvironment("Local"))
{
    // Return more error details locally
    app.UseDeveloperExceptionPage();
}
else
{
    // Use the default middleware
    app.UseExceptionHandler();
}
```

Use the `ModelState` and `ValidationProblem` members of the `ControllerBase` class to return
ProblemDetails responses:

```csharp
ModelState.AddModelError("widgetName", "widgetName cannot contain a bad word");

return ValidationProblem(ModelState);
```

And override the status code if you need to:

```csharp
ModelState.AddModelError("widgetName", "A different widget is already using this name.");

return ValidationProblem(statusCode: StatusCodes.Status409Conflict, modelStateDictionary: ModelState);
```

This additional work makes using the API much simpler since the 'problem' responses follow a
consistent format. Your users will thank you.
