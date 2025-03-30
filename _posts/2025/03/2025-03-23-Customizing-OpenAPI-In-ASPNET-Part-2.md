---
layout: post
title: 'Customizing OpenAPI Documentation In ASP.NET - Part 2'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

In Part 1 of [Customizing OpenAPI Documentation In ASP.NET]({% post_url 2025/03/2025-03-16-Customizing-OpenAPI-In-ASPNET-Part-1 %}),
we looked at how to ensure responses for each the status code were provided and that the payloads.
There are still more customizations we can make.

<!--more-->

### Customizing Using XML Comments

In .NET 8, the Swagger library can use the generated XML comments to enhance the OpenAPI documentation.

To enable the XML Comment Generation, add the following to the CSPROJ file:


```xml
<PropertyGroup>
   <GenerateDocumentationFile>true</GenerateDocumentationFile>
   <DocumentationFile>{WebProjectAssemblyName}.xml</DocumentationFile>
   <NoWarn>1701;1702;CS1591</NoWarn>
</PropertyGroup>
```

The `DocumentationFile` value should match the name of the assembly. This isn't necessary, but it
makes configuring things easier.

The `NoWarn` property suppresses the missing XML comment warnings. Alternatively, this can be
suppressed using the `editorconfig` file:

```ini
# CS1591: Missing XML comment for publicly visible type or member
dotnet_diagnostic.CS1591.severity = none
```

Now you tell the generator where to find the comments file:

```csharp
services.AddSwaggerGen(options =>
{
   var xmlCommentsFile = Assembly.GetExecutingAssembly().Location.Replace("dll", "xml");
   options.IncludeXmlComments(xmlCommentsFile);
}
```

XML comments on the controller methods will be included in the generated OpenAPI document:

XML Comment | OpenAPI Document Usage
----------- | ----------------------
summary | summary of endpoint
remarks | description of endpoint (can be formatted using Markdown syntax)
param | description of requestBody

The Swagger generator recognizes XML comments as well. Add comments to the request/response
models properties to include them in your generated document:

```csharp
   /// <summary>
   /// The identifier of the widget
   /// </summary>
   public int Id { get; set; }
```

Unfortunately, if you include `/// <remarks></remarks>` they are not included in the generated
document.

**NOTE** The built-in Microsoft OpenApi generator for .NET 9 does not recognize XML comments. However,
as of this writing, the .NET 10 Preview now supports XML comments.

### Providing Examples

Sometimes, it can be useful to override the default examples and provide your own. First, install the
`Swashbuckle.AspNetCore.Filters` package:

```bash
dotnet add package Swashbuckle.AspNetCore.Filters
```

Create a schema filter:

```csharp
public class ProblemDetailsSchemaFilter : ISchemaFilter
{
    public void Apply(OpenApiSchema schema, SchemaFilterContext context)
    {
        if (context.Type == typeof(ProblemDetails))
        {
            schema.Example = new Microsoft.OpenApi.Any.OpenApiObject
            {
                ["type"] = new Microsoft.OpenApi.Any.OpenApiString("URI reference that identifies the problem type"),
                ["title"] = new Microsoft.OpenApi.Any.OpenApiString("human-readable summary of problem"),
                ["status"] = new Microsoft.OpenApi.Any.OpenApiInteger(999),
                ["detail"] = new Microsoft.OpenApi.Any.OpenApiString("human-readable explanation"),
                ["traceId"] = new Microsoft.OpenApi.Any.OpenApiString("A trace identifier following the W3C tracing context"),
                ["errors"] = new Microsoft.OpenApi.Any.OpenApiObject
                {
                    ["$"] = new Microsoft.OpenApi.Any.OpenApiArray
                    {
                        new Microsoft.OpenApi.Any.OpenApiString("request-level error message")
                    },
                    ["property"] = new Microsoft.OpenApi.Any.OpenApiArray
                    {
                        new Microsoft.OpenApi.Any.OpenApiString("property-level error message")
                    }
                }
            };
        }
    }
}
```

Then register the filter:

```csharp
services.AddSwaggerGen(options =>
{
   options.SchemaFilter<ProblemDetailsSchemaFilter>();
}
```

### Branding the Swagger UI

You can also provide some customization to the Swagger UI:

- Custom CSS
- Custom Index.html

For example, let's say you want to hide (or replace) the Swagger logo. You can inject custom CSS into
the Index.html. First, add a `wwwroot` folder. In this folder, include a sub-folder using the same
value as the RoutePrefix you are serving your Swagger UI page from. In my case, that is `docs`. In
this folder, create a CSS file `custom.swagger.css`:

```css
#swagger-ui > section > div.topbar > div > div > a {
    display: none;
}
```

Make sure your ASP.NET pipeline is configured to server static assets:

```csharp
app.UseStaticFiles();
```

Then configure the Swagger UI to include the custom CSS file:

```csharp
app.UseSwaggerUI(options =>
{
   options.RoutePrefix = "docs";

   options.SwaggerEndpoint("openapi/v1.json", "My API");

   options.InjectStylesheet("custom.swagger.css");
});
```

If you want more control, you can also replace the entire `Index.html` file. But if you do so, it
is recommended you start with the default, for the version of Swagger UI you are using.

Create an `EmbeddedAssets` folder, and in the folder add a copy of the `Index.html` file from here:

<https://github.com/domaindrivendev/Swashbuckle.AspNetCore/blob/v6.6.2/src/Swashbuckle.AspNetCore.SwaggerUI/index.html>

**NOTE** This can be tricky if you don't choose the correct version of this file for the Nuget package
you are using. Be certain you select the correct one. In the above example, this is the `Index.html` for
version v6.6.2 of the Nuget package.

Configure this file to be an embedded resource in the CSPROJ file:

```xml
  <ItemGroup>
    <EmbeddedResource Include="EmbeddedAssets\Index.html" />
  </ItemGroup>
```

Then, you can configure the Swagger UI to use this file:

```csharp
app.UseSwaggerUI(options =>
{
    options.RoutePrefix = "docs";
    options.SwaggerEndpoint("openapi/v1.json", "My API");

    var assembly = typeof(Program).Assembly;

    options.IndexStream = ()
        => assembly.GetManifestResourceStream($"{assembly.GetName().Name}.EmbeddedAssets.Index.html");
});
```

Replacing the `Index.html` can be helpful if you want to replace the default favicons with your own branded ones.

### Summary

There are a lot of customizations you can make to the default Swagger OpenApi document and to the
Swagger UI. The XML comments are very helpful as are the custom examples. The Swagger UI is also very
customizable. Have fun!
