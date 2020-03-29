---
layout: post
title: 'Dealing with ASPNET Core 3 API Contract Changes'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

If you work with HTTP API endpoints long enough, you will encounter scenarios when you want/need to make changes to the data contract without breaking changes. Prior to ASPNET 3, the `[DataContract]` and `[DataMember]` attributes made it possible. However, with the new JSON parsing library in ASPNET 3, these are no longer as straightforward.

<!--more-->

## The Problem

Each HTTP API defines a contract for callers to consume. Often, the contract defined by external services use concepts that don't line up perfectly with the concepts in your application. Take for example the following JSON being returned from a GET Http endpoint for a customer:

```json
{
    "name": "John Doe",
    "email": "John.Doe@example.com",
    "statusId": "active"
}
```

In your application, the standard naming convention would have been to use the following:

```csharp
public class CustomerResponse
{
    public string FullName { get; set; }

    public string EmailAddress { get; set; }

    public string Status { get; set; }
}
```

When the contract exposed by the HTTP API endpoint and your preferred representation are different, you require
a mapping mechanism to control the JSON parsing.  

## Using `[DataContract]` and `[DataMember]`

Prior to ASPNET Core 3.0, JSON parsing and encoding was performed by the NewtonSoft JSON parser. To map the encoding
of the JSON to the C# code, you could use the `[DataContract]` and `[DataMember]` attributes from the `System.Runtime.Serialization` library:


```csharp
using System.Runtime.Serialization;

[DataContract]
public class CustomerResponse
{
    [DataMember(Name = "name")]
    public string FullName { get; set; }

    [DataMember(Name = "email")]
    public string EmailAddress { get; set; }

    [DataMember(Name = "StatusId")]
    public string Status { get; set; }
}
```

## Changes to ASPNET 3: `System.Text.Json`

With ASPNET, [Microsoft is including a new parser for JSON][new-library]. There are good reason for this. And to their
credit, Microsoft has included backwards compatibility so you can continue to use NewtonSoft.JSON if necessary.

Unfortunately, the default JSON serialization no longer supports the `[DataContract]` and `[DataMember]` attributes.

However, there is a new mechanism to produce the same behavior as in prior versions.

## Using `[JsonPropertyName]` and `[JsonIgnore]`

The new JSON parsing library contains its own serialization behavior attributes. `JsonPropertyName` allows you to set the 
name you want the JSON to use. `JsonIgnore` will ensure the JSON serializer does not include the property when encoding 
or decoding the JSON.

```csharp
using System.Text.Json.Serialization;

public class CustomerResponse
{
    [JsonPropertyName("name")]
    public string FullName { get; set; }

    [JsonPropertyName("email")]
    public string EmailAddress { get; set; }

    [JsonPropertyName("StatusId")]
    public string Status { get; set; }

    [JsonIgnore]
    public Guid Id { get; set; }
}
```

Notice it is not necessary to decorate the class with an attribute like `[DataContract]`. In addition, the NewtonSoft JSON serializer would exclude any property not decorated with a `DataMember` attribute if the class was decorated with `[DataContract]`. Using the new library, all properties are included, except the ones explicitly decorated wit the [JsonIgnore] attribute.

## Summary

Although the new ASPNET 3.0 JSON serialization differs from the previous implementations, Microsoft has provided a lot of backwards compatibility and mechanisms to perform the same JSON serialization activities. 

[new-library]: https://devblogs.microsoft.com/dotnet/try-the-new-system-text-json-apis/
