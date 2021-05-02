---
layout: post
title: 'Reusing Object in Unit Tests'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Writing unit tests sometimes involves creating complex objects to test with. Reuse these object carefully.

<!--more-->

### The Problem

Let's suppose you construct a test case like this:

```csharp
var customerInRepository = new Customer { Id = 1, ... };

var mockRepository = new Mock<ICustomerRepository>();
mockRepository
    .Setup(x => x.GetByIdAsync(1), CancellationToken.None)
    .ReturnAsync(customerInRepository);

var underTest = new CustomerService(mockRepository.Object);

var updatedCustomer = new Customer { Id = 1, ... };

var response = underTest.UpdateCustomerAsync(1, updatedCustomer, CancellationToken.None);

Assert.True(response);

mockRepository.Verify(x => x.CommitAsync(CancellationToken.None), Times.Once);
```

What is being tested/asserted isn't the part I'm focusing on. But rather the fact that the object in the repository (`customerInRepository`) and the updated data being passed in (`updatedCustomer`) are different objects.

This is how I would expect tests to be written. But I have been guilty of, and have seen others as well, taking shortcuts when it comes to large complex objects.

If the "customer" class is a complex graph, it sometimes can be a lot of effort to construct one in the test case. It can be tempting to create one of them and use it in both cases. But that is not a good idea.

It can create two problems for you in the long run. First, it can create a false-negative. Reusing the objects in this manner can make it appear as if the test case passed but in reality there is a bug in the code that goes undetected. This is because the updated data and the data the code is attempting to update start off their life looking exactly the same. There is no way for the assertions made after the fact to know if the final state of the object is due to the code that was executed, or due to the fact the object looked the same going in as it did going out.

The second issue that can appear is broken tests when things change. This happened to me recently. The code to update an object changed and it caused a completely unrelated unit test to fail. The failing test reused an object for both the data in the repository and the updated data being passed into the function being tested. There was no real issue, no regression that introduced a bug. But rather, the test failed because it modified both instances of the object when it should have only modified one of them.

### The solution

The solution was to create separate objects. 

```csharp
var customerInRepository = CreateTestCustomer(1);

var mockRepository = new Mock<ICustomerRepository>();
mockRepository
    .Setup(x => x.GetByIdAsync(1), CancellationToken.None)
    .ReturnAsync(customerInRepository);

var underTest = new CustomerService(mockRepository.Object);

var updatedCustomer = CreateTestCustomer(1);

var response = underTest.UpdateCustomerAsync(1, updatedCustomer, CancellationToken.None);

Assert.True(response);

mockRepository.Verify(x => x.CommitAsync(CancellationToken.None), Times.Once);
```

When constructing unit tests, you can avoid this problem using helper functions. Create a factory function to construct objects. You can pass in arguments to the function to create specific scenarios for tests. Then, use separate calls to these functions to create the objects used in your tests. This avoids the reuse of objects and the test cases will be better for it.
