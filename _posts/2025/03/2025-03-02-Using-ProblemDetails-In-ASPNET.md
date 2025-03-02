---
layout: post
title: 'Using ProblemDetails In ASP.NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I wrote about using ProblemDetails a while back:

[Problem Details with ASP.NET Core]({% post_url /2021/04/21-04-18-Problem-Details-With-ASPNET-Core %})  

Since then, they have become a first-class citizen in the ASP.NET libraries and can be added as
follows:

<!--more-->

```csharp
builder.Services.AddProblemDetails();

builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();

app.Run();
```

Here is the default response using ProblemDetails for a 404 Not Found response:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Not Found",
  "status": 404,
  "traceId": "00-5c237de4c4be40cd28709c282af2c3c3-984c1ab16ce367e2-00"
}
```

For a 400 Bad Request response, we receive:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": [
      "The Name field is required."
    ]
  },
  "traceId": "00-6328f85c9aae667e240df8ef3e56e06c-9f053f99efa0903a-00"
}
```

And for 500 errors response, we receive no response body at all.

### The Goal

What I would like to do is make the responses more consistent:

1. Return a ProblemDetails response for 500 Internal Server Error responses
2. Include an "errors" property with every response.
3. Provide additional detail to callers to help them interpret the responses

### ProblemDetails for 500 Internal Server Error Responses

To enable this feature, we use the new feature added in .NET 8:

```csharp
app.UseExceptionHandler();
```

Now the 500 Error response contains a payload body:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.6.1",
  "title": "An error occurred while processing your request.",
  "status": 500
}
```

This response doesn't include the `traceId` property, which is very important when trying to locate
log entries related to the particular request. To add this value, we need to customize the response
for the ProblemDetails. We can do that via an overloaded version of the `AddProblemDetails()`
method:

```csharp
builder.Services.AddProblemDetails(options =>
    options.CustomizeProblemDetails = (context) =>
    {
        // include traceId with every response
        if (!context.ProblemDetails.Extensions.ContainsKey("traceId"))
        {
            var traceId = Activity.Current?.Id ?? context.HttpContext.TraceIdentifier;
            if (!string.IsNullOrWhiteSpace(traceId))
            {
                context.ProblemDetails.Extensions["traceId"] = traceId;
            }
        }
    });
```

And as you can see, we now get the `traceId` property included.

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.6.1",
  "title": "An error occurred while processing your request.",
  "status": 500,
  "traceId": "00-f85e17e3aa994829e1afa80254f961a0-c7228d160146bb05-00",
}
```

### Adding Extensions

We want to ensure all ProblemDetails include an "errors" property. This makes it easier for
users of the API to parse the problem details, since they won't have to check if the errors
are provided or not. To add this property on every response, we can add the following to our
customizations:

```csharp
// include errors object if it doesn't exist
if (!context.ProblemDetails.Extensions.ContainsKey("errors"))
{
    context.ProblemDetails.Extensions["errors"] = new Dictionary<string, string[]>();
}
```

### Adding Details

In our context, we want users of our API to have some additional details regarding the problems
they have encountered:

404 - let them know that the missing resource may be due to it having expired.
500 - let them know that they should provide the `traceid` if they contact us for more details

We can add those customizations too:

```csharp
switch (context.ProblemDetails.Status)
{
    case StatusCodes.Status404NotFound:
        context.ProblemDetails.Detail = "The requested resource does not exist, or it may have expired";
        break;
    case StatusCodes.Status500InternalServerError:
        context.ProblemDetails.Detail = "Please provide the 'traceid' when contacting us.";
        break;
}
```

This will result in the detail being included:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.6.1",
  "title": "An error occurred while processing your request.",
  "status": 500,
  "detail": "Please provide the 'traceid' when contacting us.",
  "traceId": "00-20a88009fbc3e3edda09e0ef06407402-27d7707b3ffeea94-00",
  "errors": {}
}
```

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.5",
  "title": "Not Found",
  "status": 404,
  "detail": "The requested resource does not exist, or it may have expired",
  "traceId": "00-b67d8279e4536501ae5a2595f33547a5-c901019057cad7e9-00",
  "errors": {}
}
```

### Summary

I hope I've demonstrated how to customize the ProblemDetails used by ASP.NET. The more we can make
these 'problem' responses easier to consume by the users of our API, the better it is for everyone.
