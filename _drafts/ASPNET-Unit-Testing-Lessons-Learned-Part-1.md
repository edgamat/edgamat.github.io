---
layout: post
title: 'ASPNET Core Unit Testing Lessons Learned Part 1'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When building a Web API endpoint with ASPNET Core, my team and I have strived to include appropriate
test coverage on all the REST endpoints. This includes unit tests on all the business/domain classes
the endpoints use to process each request. This is the first in a series of posts that describes some
of the lessons we have learned along the way.

<!--more-->

## Contract Mappings

One area where we had a lot of regression bugs crop up is the code used to translate incoming JSON
payloads to domain classes. Everything would look okay until we ran an integration test and then
we'd find that a value in the incoming JSON payload (say for a POST or PUT endpoint) would not
be present in the mapped domain object. We call the incoming JSON payload the Contract Model.

This was an issue worth creating explicit unit tests around because it would cause a lot of 
waste. The missing values were difficult to catch and usually only caught far after the code 
changes had been made. And sometimes the missing data was due to errors in the business logic,
not the translation code.

Our solution for this issue was to add unit tests on the mapping logic and test that each property
in the domain object contained a non-default value coming out of the mapping function that converted
the a contract model object into a domain object. Once we had these tests passing for all existing properties,
here is the process we used to make changes:
- Modify the contract model to reflect the new changes
- Add a new property to the domain class, or change the name of an existing one
- Modify the unit tests to expect the new shape of the domain class to have non-default values
- Watch the unit tests fail for the expected reasons due to the changes
- Modify the mapping function to make the unit tests pass

Here's and example. Let's say the incoming Contract Model is as follows:

```csharp
[DataContract]
public class UpdateContractRequestModel
{
    [DataMember(Name = "id")]
    public Guid Id { get; set; }

    [DataMember(Name = "name")]
    public string Name { get; set; }

    [DataMember(Name = "mailedOn")]
    public DateTime MailedOn { get; set; }
}
```

And the corresponding domain model is this:

```csharp
[DataContract]
public class UpdateCommand
{
    public Guid Id { get; set; }

    public string Name { get; set; }

    public DateTime MailedOn { get; set; }
}
```

The translation of the incoming data is a function:

```csharp
public UpdateCommand CreateCommand(UpdateContractRequestModel model)
{
    return new UpdateCommand
    {
        Id = model.Id,
        Name = model.Name
    };
}
```

The bug in the function is that the `MailedOn` date is not mapped and the resulting domain object
is set to the default value.

We constructed unit tests to catch these errors:

```csharp
[Fact]
public void EnsureTranslationIsCorrect()
{
    var sut = CreateTestSubject();
    
    var model = new UpdateContractRequestModel { Id = Guid.NewGuid();, Name = "test", MailedOn = DateTime.Today };
    
    var result = sut.CreateCommand(model);
    
    Assert.Equal(model.Id, result.Id);
    Assert.Equal(model.Name, result.Name);
    Assert.Equal(model.MailedOn, result.MailedOn);
}
```

Granted, this is a limited example. However, imagine where the contract model and domain object are significantly different
in their shape. Imagine where the number of properties on the classes that need to be mapped is in the dozens. This was
where we relied on the test cases to catch these mapping issues.

### Dealing with False Negatives

In some cases, the unit test would expect to fail but would pass. This was due to how the assertions were dealing with `null` values.
If the incoming model had a null value for a property, the domain object would also have a null value. To avoid this, we 
began using explicit values if a null was present in the domain object:

```csharp
[Fact]
public void EnsureTranslationIsCorrect()
{
    var sut = CreateTestSubject();
    
    var model = new UpdateContractRequestModel { Id = Guid.NewGuid();, Name = null, MailedOn = DateTime.Today };
    
    var result = sut.CreateCommand(model);
    
    Assert.Equal(model.Id, result.Id);
    Assert.Equal(model.Name, result.Name ?? "MISSING");
    Assert.Equal(model.MailedOn, result.MailedOn);
}
```
Now if the mapping function under test changes and the `Name` property is not set, we'll be notified.

## Issues and Observations

As a lesson learned, we discovered that these mapping functions needed full test coverage to protect us from regression
errors appearing in the code. I would keep this in mind for any future project. 

We used helper functions (like `AutoMapper`) sometimes to make the mapping functions easier to maintain. This did not
change the need for these unit tests. Regardless of how the mapping is achieved, including the mapping tests helped
us catch several regression bugs. 


