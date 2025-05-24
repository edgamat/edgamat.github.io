---
layout: post
title: 'Integration Tests with ASP.NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I have seen a lot of mentions recently for writing integration tests with ASP.NET. I wanted to see
what that experience was like.

<!--more-->

### Beyond Unit Tests

Unit tests are a powerful way to help understand if your code is working correctly. But they aren't
a complete picture. You typically won't know how well the different units of code work together. For
that you need different types of tests. One such type is integration tests.

You can find a lot of different ways to describe integration tests. At their core, integration tests
are tests to confirm that different components in your code work together as expected. A simple case
is when you have two classes or a suite of classes working together in a test. Another case is when
you have different modules in your code working together. In this context, you are not creating an
instance of one thing and creating mock implementations of the others. That, I would argue, is a unit
test. Rather, the components working together are the actual implementations.

That is not to say you don't use mock implementations when integration testing. For example, you may
want to test that 2 components work correctly with one another. Each of these components depend on
a database (accessed via EF Core, for example). You may wish to use a mock DB Context for these tests
and I would still consider these integration tests.

### Integration Tests with ASP.NET

Writing integration tests with ASP.NET can help you verify that the entire request pipeline is working
correctly. We can test things just like they would be in a real environment. These tests can help
confirm that the routing, middleware, authentication, model binding, filters and endpoints/controllers
all work together as expected.

What do these tests look like? They are based on making HTTP requests using the standard `HttpClient`
class:

```csharp
[Fact]
public async Task GetById_Should_Return_NotFound_If_Widget_Does_Not_Exist()
{
    var client = _factory.CreateClient();

    var response = await client.GetAsync("/widgets/9999");

    // check that the response is not successful
    Assert.Equal(System.Net.HttpStatusCode.NotFound, response.StatusCode);
}
```

### Enable Integration Testing

As with most things, the first thing we need to do is install a NuGet package to our test project. In
this example, I'll be using `xUnit`. The package to add is `Microsoft.AspNetCore.Mvc.Testing`:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Testing
```

Note, if you are not using the latest version of .NET, you may need to explicitly set the version
of the package you want (e.g. `--version 8.0.16`).

Next we need to tweak the SDK moniker for the xUnit test project. Open the test project CSPROJ file
and change the moniker from:

```xml
<Project Sdk="Microsoft.NET.Sdk">
```

to

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

The last thing we need to do is make the `Program` class in the web project accessible to the tests.
To do this, add the following line to your `Program.cs` file in your web project:

```csharp
// Program.cs
public partial class Program { }
```

### The WebApplicationFactory Class

The NuGet package we added includes a factory class that makes all this possible. The `WebApplicationFactory`
class will spin up a working version of the website and give us an `HttpClient` ready to make requests
to this instance.

```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            // Remove or replace services for testing, e.g., replace DB context
        });

        builder.UseEnvironment("Local");
    }
}
```

In `xUnit`, you can inject the factory into your tests using a test fixture:

```csharp
public class WidgetTests : IClassFixture<CustomWebApplicationFactory>
{
    private readonly CustomWebApplicationFactory _factory;

    public WidgetTests(CustomWebApplicationFactory factory)
    {
        _factory = factory;
    }

    [Fact]
    ..........
}
```

### Writing Integration Tests

As previously mentioned, these integration tests are based on making HTTP requests to your API. This
allows you to leverage all your knowledge of calling HTTP endpoints from within your code.

Here is a test for a GET endpoint:

```csharp
[Fact]
public async Task GetById_Should_Return_Success_If_Widget_Exists()
{
    var client = _factory.CreateClient();

    var response = await client.GetAsync("/widgets/1005");

    response.EnsureSuccessStatusCode();
}
```

And here is one for a POST endpoint:

```csharp
[Fact]
public async Task Post_Should_Return_Success_And_Location_Header()
{
    var client = _factory.CreateClient();

    var newWidget = new
    {
        Name = "Integration Test Widget",
        Description = "Integration Test Widget"
    };

    var response = await client.PostAsJsonAsync("/widgets", newWidget);

    // check that the response is successful
    Assert.Equal(System.Net.HttpStatusCode.Created, response.StatusCode);

    Assert.NotNull(response.Headers.Location);
}
```

### What Should Be Asserted?

The goal of the integration tests is to verify _observable_ behavior only. But there is a lot you
could be checking with the response that is returned from your HTTP request. What should we be testing?

1. **Assert on the expected status code** - checking this value confirms that the model binding, routing,
validation and authentication/authorization are working correctly. A 400 might mean that the model
binding or validation is wrong. A 404 might indicate a problem with the routing. A 401/403 could mean
the security configuration is incorrect. All this can be confirmed from this single check.

```csharp
Assert.Equal(System.Net.HttpStatusCode.Created, response.StatusCode);
```

2. **Assert the Content Type** - We want to ensure that the correct content type is returned. This
can be useful when checking for a 'special' content type (like `application/problem+json`):

```csharp
Assert.Equal(System.Net.HttpStatusCode.BadRequest, response.StatusCode);

Assert.Equal("application/problem+json; charset=utf-8", response.Content.Headers?.ContentType?.ToString());
```

3. **Assert on the Response Body** - It is important to check that the body of the response contains
what you expect. For example, you can check that the body is not empty and contains JSON of the proper
shape.

```csharp
var j = await response.Content.ReadFromJsonAsync<WidgetResponse>();
Assert.True(j is not null);
```

4. **Assert Data Integrity** - We should confirm that the correct data is in the response body:

```csharp
Assert.True(json.Id > 0);
Assert.True(json.CreatedAt != default);
```

These are tricky assertions. If you assert too much, the test can be very brittle. If you assert too
little you may miss important changes that should fail a test. My advice is to test as little as
possible to confirm the desired behavior.

5. **Assert Error Messages** - We should ensure the error messages are returned as expected. However,
you shouldn't be testing the framework works as expected. For example, we shouldn't assert that the 
standard problem details contains a bad request 400 error message for a required field. But we should
ensure that any custom validation is done properly:

```csharp
var problem = await response.Content.ReadFromJsonAsync<ValidationProblemDetails>();
Assert.NotNull(problem);
Assert.Equal("Widget with name Integration Test Widget already exists.", problem.Detail);
```

### Summary

Writing integration tests is important do understand if your code is going to work when all the component
are working together. With the `WebApplicationFactory` you can test your actual web application by
making HTTP requests toa version running in memory. These tests can really give you confidence that
the request pipeline is configured properly and that the observable behavior is correct.