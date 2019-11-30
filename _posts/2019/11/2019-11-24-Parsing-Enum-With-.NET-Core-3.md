---
layout: post
title: 'Parsing Enum Values with ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When you choose to use an enumeration in an ASP.NET incoming model, it is often desirable to convert
the numerical representation to a string. The changes to how JSON is parsed in ASP.NET Core 3 means
the way this parsing is done has changed.

<!--more-->

The scenario I am referring to is when you have a class such as this:

```csharp
[DataContract]
public enum UserStatus
{
    Unknown,

    Active,

    Inactive,

    Locked
}

[DataContract]
public class CreateUserDto
{
    [DataMember(Name = "status")]
    public UserStatus Status { get; set; }

    [DataMember(Name = "emailAddress")]
    public string EmailAddress { get; set; }
}
```

It serves as an input model to an API endpoint for creating a user:

```csharp
[HttpPost]
public IActionResult Create(CreateUserDto user)
{
    var newUser = _service.CreateNewUser(user);

    return CreatedAtAction(nameof(Get), newUser);
}
```

With .NET Core 2.x, the incoming JSON can use the numeric or string value for the Status. In other
words, the .NET Core model binder will parse these two payloads the same way:

```json
{
    "status": "Inactive", "emailAddress": "test@example.com"
}

{
    "status": 2, "emailAddress": "test@example.com"
}
```

This is possible due to the way the built-in JSON Parser ([Newtonsoft.Json][newtonsoft]) handles enumerations.

With ASP.NET Core 3.0, Microsoft decided to stop using Newtonsoft.Json in favor of it's own
custom JSON parser. The reasons for the change can be found here: [The future of JSON in .NET Core 3.0][the-future-of-json].

Using the same example above, the default behavior has changed. Now, the string representation of the
enumeration is no longer parsed and the system returns a 400 Bad Request response if this is attempted:

```json
{
  "errors": {
    "$.status": [
      "The JSON value could not be converted to EnumParsing.Api.Users.UserStatus. Path: $.status | LineNumber: 1 | BytePositionInLine: 24."
    ]
  }
}
```

The solution to providing this parsing capability is to explicitly add the conversion logic during startup.

In the `ConfigureServices` routine of the `Startup` class, add the following:

```csharp
services.AddMvc()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
    });
```

Now the code will parse the string value or the numerical value, just as it did
in ASP.NET Core 2.x.

That isn't the end of the story however. If you don't want to use the new JSON parser
in ASP.NET Core 3, the designers of the framework has provided a means to continue to
use the Newtonsoft.Json parser. Install the following NuGet package:

`Microsoft.AspNetCore.Mvc.NewtonsoftJson`

Then, replace the `AddJsonOptions` used previously with the following:

```csharp
services.AddMvc()
    .AddNewtonsoftJson(json =>
    {
        json.SerializerSettings.Converters.Add(new Newtonsoft.Json.Converters.StringEnumConverter());
    });
```

Parsing enumerations provided the string representation can be desirable and it is still possible
to do so with the new JSON parser in ASP.NET Core 3. And if you prefer, you can still use the
previous parser, Newtonsoft.Json.

[newtonsoft]: https://www.newtonsoft.com/json
[the-future-of-json]: https://github.com/dotnet/announcements/issues/90
