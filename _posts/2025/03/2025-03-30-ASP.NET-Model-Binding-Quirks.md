---
layout: post
title: 'ASP.NET Model Binding Quirks'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Awhile ago I wrote about using the new JSON library and model binding:
[Dealing with ASPNET Core 3 API Contract Changes]({% post_url /2020/03/2020-03-29-Dealing-With-ASPNET-Core-3-API-Contract-Changes.md %}).
I only recently started to use it with ASP.NET 8. It has some quicks... let me explain.

<!--more-->

### Background

This week a co-worker reached out to me about some odd behavior he was experiencing. He purposely
left out a required property in a JSON payload. He was expecting a 400 BAD REQUEST response that
included model state errors like this:

```json
  "errors": {
    "description": [
      "The description field is required."
    ]
  }
```

But instead he got this:

```json
  "errors": {
    "$": [
      "JSON deserialization for type 'ModelBindingDemo.CreateWidgetRequest' was missing required properties including: 'description'."
    ]
  }
```

### Default Behavior

In ASP.NET, you can declare a model class and bind it to incoming JSON payloads. For
example, here is a class used to bind to a JSON payload used to create a widget:

```csharp
public class CreateWidgetRequest
{
    [JsonPropertyName("name")]
    public string Name { get; init; } = null!;

    [JsonPropertyName("description")]
    public string Description { get; init; } = null!;

    [JsonPropertyName("available_on")]
    public DateOnly AvailableOn { get; init; }

    [JsonPropertyName("quantity")]
    public int Quantity { get; init; }
}
```

Then, in the controller, you declare the 'Create' method like this:

```csharp
[HttpPost]
public ActionResult<WidgetResponse> Create(CreateWidgetRequest request)
{
}
```

By default, the ASP.NET framework will parse the JSON payload and use the data to construct a class
of type `CreateWidgetRequest`.

If you are using nullable reference types (typically this is enabled by default), the model binder
will treat missing values as a validation error. For example, if you don't include a `description`
property, the response will be:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "description": [
      "The description field is required."
    ]
  },
  "traceId": "00-1fcd4545b00e62b3cf5c205f26ebd3f2-7b95f7e67c4da655-00"
}
```

This is what my coworker (and I) were expecting to occur. Why did we get a JSON parsing error?

### Required Properties

In .NET 8 we started using the new `required` keyword on our API contract models. The `required` modifier
indicates that the field or property it's applied to must be initialized by an object initializer.

When I looked at the code, `required` was applied to all the properties:

```csharp
public class CreateWidgetRequest
{
    [JsonPropertyName("name")]
    public required string Name { get; init; }

    [JsonPropertyName("description")]
    public required string Description { get; init; }

    [JsonPropertyName("available_on")]
    public required DateOnly AvailableOn { get; init; }

    [JsonPropertyName("quantity")]
    public required int Quantity { get; init; }
}
```

When the model binder is constructing a new instance of the `CreateWidgetRequest` object, it uses
the `Deserialize` method of the `JsonSerializer` class in the `System.Text.Json` library. This
method is what is generating the JSON parsing error we see in the response. It treats the missing
'required' fields as errors and won't deserialize a new object if any properties with
the `required` modifier are missing.

Now we know why we are receiving the JSON parsing error. Excellent. But what can we do about it?

### Should the Required Modifier Be Avoided?

Within each project (or team or company) you have to make decisions based on your context. If you
feel strongly about the `required` modifier, then you have to accept that missing properties will
generate a JSON parsing error.

Including the `required` modifier does have some nice behaviors you may not want to get loose. For
example, it can alert you that `struct` properties (like `DateOnly` and `int`) are missing.
This wasn't possible in the past. You would need to declare the property as nullable and then use
the `Required` attribute to ensure the value was provided in the incoming payload:

```csharp
[Required]
[JsonPropertyName("available_on")]
public DateOnly? AvailableOn { get; init; }
```

The downside to this way of validating the request is that it represents the data incorrectly once
you are using the constructed object. For example, in your controller, you now have to deal with
the nullable data type:

```csharp

// Use the `GetValueOrDefault()` ...
var widget = new Widget
{
    Id = Guid.NewGuid(),
    Name = request.Name,
    Description = request.Description,
    AvailableOn = request.AvailableOn.GetValueOrDefault(), // <----
    Quantity = request.Quantity
};

// Or, use the `!` operator
var widget = new Widget
{
    Id = Guid.NewGuid(),
    Name = request.Name,
    Description = request.Description,
    AvailableOn = request.AvailableOn!.Value, // <----
    Quantity = request.Quantity
};
```

We are basically telling the compiler a lie (that the `AvailableOn` property might be `null`) and
then we have to deal with that lie using a bunch of un-necessary syntax in the code. From a coding
perspective, I prefer using the `required` modifier and avoid using nullable properties with the
`Required` attribute. But, JSON parsing errors aren't great to show users.

### Can't I have the Best of Both Worlds?

Let's assume that we don't want to include the `required` modified on the properties. That solves
our behavior problem with the `description` property (and other reference types). But it leaves us
with a problem with the value data types, like `DateOnly` and `int`.

One way to modify this behavior is to set the value type properties to an invalid value, by default:

```csharp
[JsonPropertyName("available_on")]
public DateOnly AvailableOn { get; init; } = DateOnly.MinValue;

[JsonPropertyName("quantity")]
public int Quantity { get; init; } = int.MinValue;
```

In our context of creating a new widget, neither of these values are what would be considered valid.
The JsonSerializer.Deserialize() method will use these values if the JSON payload does not include
the properties. We can then introduce a `Range` attribute to validate the inputs:

```csharp
[Range(typeof(DateOnly), "0001-01-02", "9999-12-31", ErrorMessage = "The field {0} is required and must be between {1} and {2}.")]
[JsonPropertyName("available_on")]
public DateOnly AvailableOn { get; init; } = DateOnly.MinValue;

[Range(typeof(int), "-2147483647", "2147483647", ErrorMessage = "The field {0} is required and must be between {1} and {2}.")]
[JsonPropertyName("quantity")]
public int Quantity { get; init; } = int.MinValue;
```

Both 'maximum' values in the `Range` attributes equal the `MaxValue` for the respective data types.
Both 'minimum' values are one value greater than the `MinValue`, which places the `MinValue` as the
only value in the invalid range.

Now if you don't include a value for the `available_on` you will see this, rather than a JSON parsing
error:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "available_on": [
      "The field available_on is required and must be between 1/2/0001 and 12/31/9999."
    ]
  },
  "traceId": "00-9fd708720cb122e673e61f73190bf6d1-a7727a1f15d15774-00"
}
```

If I provided an empty JSON payload, I would see no JSON parsing errors:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "name": [
      "The name field is required."
    ],
    "quantity": [
      "The field quantity is required and must be between -2147483647 and 2147483647."
    ],
    "description": [
      "The description field is required."
    ],
    "available_on": [
      "The field available_on is required and must be between 1/2/0001 and 12/31/9999."
    ]
  },
  "traceId": "00-145da630023d95181678a8f99278df17-f92856a313a6567f-00"
}
```

### Problem Solved?

Even using this approach to avoid using the `required` keyword, it doesn't mean I won't get JSON
parsing errors. What if I provide invalid dates or numbers?

```json
{
  "name": "My Widget",
  "description": "This is a test widget",
  "available_on": "invalid date",
  "quantity": 10
}
```

This will still result in a JSON parsing error, because it cannot convert the value "invalid date"
to an instance of `DateOnly`:

```json
"The JSON value could not be converted to System.DateOnly. Path: $.available_on | LineNumber: 3 | BytePositionInLine: 32."
```

The same result will occur if `null` is explicitly provided in the payload:

```json
{
  "name": "My Widget",
  "description": "This is a test widget",
  "available_on": null,
  "quantity": 10
}
```

In the end, we are still at the mercy of what the `JsonSerializer` does with the payload.

### In My Context, I Choose...

As I said earlier, everyone's context is different. Regardless of which approach you take, there will
be tradeoffs. Make the best decision for your context.

For me personally, I'm torn. I prefer the simple nature of the non JSON paring errors. I think these
are much easier for the user to consume. But the `required` modifier makes it much easier to indicate
which fields in the payload are required. And the behavior is consistent, regardless of the data type (works
the same for both reference and value types).

I experimented with a custom `TextInputFormatter` to override the default JSON input formatter to
see if I could improve the JSON parsing messages. In the end, I was able to see some improvements,
but it still didn't handle all the scenarios. I might come back to it one day.

In the meantime, since we are using a JSON parser, we should expect to generate JSON parsing
messages. I think using the `required` modifier has more benefits than drawbacks. I think this would
be my default approach for new projects. The key to this approach is communicating the expected
behavior to anyone using your API. Here is an example:

- If you submit a JSON payload with missing or invalid data, expect a JSON parsing error message.
- The data types for each property and whether they are required or not can be found in the
  published OpenAPI document for the API (provide people with a link to the document).
- `null` is not a valid value for a required property.
- dates (`DateOnly`) should be provided in the form 'YYYY-MM-DD'
- timestamp instances (`DateTime`) should be provided in UTC form: 'YYYY-MM-DDThh:mm:ssZ'

### Summary

None of these approaches are without their drawbacks. But the key is consistency. No caller of the
API should be surprised when they submit a payload. Make sure you communicate how the API behaves
when data is invalid or missing. In the end, that's the best advice to follow.
