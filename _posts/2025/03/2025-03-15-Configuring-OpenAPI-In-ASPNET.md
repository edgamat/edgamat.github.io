---
layout: post
title: 'Configuring OpenAPI In ASP.NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Providing documentation to callers of an API can be extremely beneficial. It can mean the difference
between an easy onboarding experience or frustration and a lot of trial and error.

<!--more-->

### OpenAPI Documentation

OpenAPI is a specification language for HTTP APIs that defines the structure and syntax of the
endpoints. It is programming language independent and is supported by many frameworks and open-source
tools. In ASP.NET, there are Nuget packages that can be used to generate OpenAPI specification
documents as well as user interfaces (a web page) to navigate the endpoints.

### Generate OpenAPI Documents

For several releases, ASP.NET has included a Nuget package called `Swashbuckle.AspNetCore` to generate
an OpenAPI document from the source code:

```bash
dotnet add package Swashbuckle.AspNetCore
```

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1");
});

var app = builder.Build();

app.UseSwagger();
```

The generated document can be accessed when the application is running:

`https://{host}/swagger/v1/swagger.json`

You can change the URL to better suit your context. The `v1` portion of the URL is called the document
name. You can change the name when generating the document:

```csharp
// documentName = "demo"
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("demo", new OpenApiInfo { Title = "My API", Version = "v1" });
});
```

And if you want to further customize the URL, you can do so using the `RouteTemplate` option:

```csharp
app.UseSwagger(options => options.RouteTemplate = "docs/openapi/{documentName}.json");
```

Now the document is found here:

`https://{host}/docs/openapi/demo.json`

### Open API in .NET 9

With .NET 9, ASP.NET now includes a new package from Microsoft to generate the OpenAPI document:

```bash
dotnet add package Microsoft.AspNetCore.OpenApi
```

```csharp
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddOpenApi()

var app = builder.Build();

app.MapOpenApi();
```

By default, the document is accessible at the following URl when the application is running:

`https://{host}/openapi/v1.json`

It too can be customized:

```csharp
// documentName = "demo"
builder.Services.AddOpenApi("demo", options =>
{
    options.AddDocumentTransformer((document, _, _) =>
    {
        document.Info = new() { Title = "My API", Version = "v1" };
        return Task.CompletedTask;
    });
});

app.MapOpenApi("docs/openapi/{documentName}.json");
```

Now the document is found here:

`https://{host}/docs/openapi/demo.json`

### Summary

Including OpenAPI documentation for an ASP.NET application is possible using Nuget packages to
generate the document from the code. The libraries give you a lot of control how the documents
are named, and how they are accessed when the application is running. But this is just a smart part
of what you can customize. We'll continue looking at further customizations in the next post.
