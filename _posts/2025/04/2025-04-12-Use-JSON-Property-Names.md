---
layout: post
title: 'Use JSON Property Names in ASP.NET Validation Error Messages'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

ASP.NET Web API projects using controllers have a powerful validation mechanism that provides callers
with detailed messages when they have sent invalid payloads. One of the problems though, is that
the messages don't use JSON Property Names if the model properties are decorated with
`[JsonPropertyName]` attributes. Let's fix that.

<!--more-->

### Background

Support you have an ASP.NET API controller that allows callers to create widgets. The incoming JSON
payload contains 3 values:

```json
{
  "widgetName": "Best Cog",
  "widgetDescription": "The best cog ever",
  "available": "2025-05-01"
}
```

The class we bind the payload to is defined as:

```csharp
public class CreateWidgetRequest
{
    [MaxLength(5)]
    [JsonPropertyName("widgetName")]
    public required string Name { get; init; }

    [MaxLength(10)]
    [JsonPropertyName("description")]
    public required string Description { get; init; }

    [JsonPropertyName("available")]
    public DateTimeOffset? AvailableOn { get; init; }
}
```

By default, the `JsonPropertyName` values are ignored if a validation error is returned to the caller:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "detail": "Please provide the 'traceid' when contacting us.",
  "errors": {
    "Name": [
      "The field Name must be a string or array type with a maximum length of '5'."
    ],
    "Description": [
      "The field Description must be a string or array type with a maximum length of '10'."
    ]
  },
  "traceId": "00-f213f7b9bdc21f7775d5b383aef10f44-b33458f31503f7d8-00"
}
```

`Name` and `Description` are used, rather than `widgetName` and `widgetDescription`. The reason the
`JsonPropertyName` attributes are used is to allow the model class to evolve and change over time
without breaking the contract with the caller. The `JsonPropertyName` values represent that
contract. We don't want to use the backing property names because they may change over time.

### The Easy Part

To make the model binder use the `JsonPropertyName` values, the framework includes a validation
metadata provider that checks for the attributes when generating the error messages. To use this
provider, change the code from:

```csharp
builder.Services.AddControllers();
```

to:

```csharp
using Microsoft.AspNetCore.Mvc.ModelBinding.Metadata;

builder.Services.AddControllers(configure =>
{
    configure.ModelMetadataDetailsProviders.Add(new SystemTextJsonValidationMetadataProvider());
})
```

Now the errors look correct:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.21",
  "title": "One or more validation errors occurred.",
  "status": 422,
  "instance": "/api/widgets",
  "errors": {
    "widgetName": [
      "The field widgetName must be a string or array type with a maximum length of '5'."
    ],
    "widgetDescription": [
      "The field widgetDescription must be a string or array type with a maximum length of '10'."
    ]
  },
  "traceId": "00-4af11f41cb6b0d8468e868dae13cc2df-61c5b21c6e9b6e55-00",
}
```

### Custom Validation Errors

When you need to add a custom error message, you will typically write the messages like this:

```csharp
if (_widgetService.GetWidgets().Any(x => x.Name.Equals(request.Name, StringComparison.OrdinalIgnoreCase)))
{
    ModelState.AddModelError(nameof(request.Name), "A different widget is already using this name.");
    return ValidationProblem(ModelState);
}
```

Use `nameof(request.Name)` as the key will result in the message using the class property name, not
the `JsonPropertyName` value.

But we can create an extension method to do what we need:

```csharp
public static void AddModelError<TModel>(
    this ModelStateDictionary modelState,
    string propertyName,
    string errorMessage)
{
    var property = typeof(TModel).GetProperty(propertyName, BindingFlags.Public | BindingFlags.Instance);
    var modelKey = property?.GetCustomAttribute<JsonPropertyNameAttribute>()?.Name ?? propertyName;

    modelState.AddModelError(modelKey, errorMessage);
}
```

Using this method is very similar to the original:

```csharp
// Original
ModelState.AddModelError(nameof(request.Name), "A different widget is already using this name.");

// New extension method
ModelState.AddModelError<CreateWidgetRequest>(nameof(request.Name), "A different widget is already using this name.");
```

This is an easy change and makes custom validation messages behave like the built-in validation error
messages. Yay!
