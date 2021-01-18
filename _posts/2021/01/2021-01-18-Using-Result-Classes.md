---
layout: post
title: 'Using Result Classes'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Over and over I have found cases where returning a single value from a function is not enough. Enter the _Result_ class.

<!--more-->

### The Result Class

The Result class represents the result of processing a request. It indicates if the processing was successful or not. And it returns either the output of the processing or a representation of the error (or errors) that occurred during processing.

The interface might look something like this:

```csharp
public interface IResult
{
    bool IsSuccess { get; }
    string[] Errors { get; }
}
```

- When the method is successful, the `IsSuccess` property is `true` and the `Errors` property is `null`.
- When the method fails, the `IsSuccess` property is `false` and the `Errors` property contains one or more error messages.

Here is the start of just such a class. 

```csharp
public class Result
{
    public bool IsSuccess { get; }
    public string[] Errors { get; }

    public Result(List<string> errors) : this(false, errors?.ToArray())
    {
    }

    public Result(bool isSuccess, string[] errors)
    {
        if (isSuccess && errors?.Any() == true)
            throw new InvalidOperationException("Can't be successful and have errors");

        if (!isSuccess && errors?.Any() == false)
            throw new InvalidOperationException("Must provide errors when failing");

        IsSuccess = isSuccess;
        Errors = errors;
    }
}
```

Here is an example of using this approach:

```csharp
    public Result InsertBook(string isbn, string title)
    {
        var errors = new List<string>();

        if (string.IsNullOrWhiteSpace(isbn))
            errors.Add("missing_isbn");

        if (string.IsNullOrWhiteSpace(title))
            errors.Add("missing_title");

        if (errors.Any())
            return new Result(errors);

        // Do insert book processing

        return new Result(true, null);
    }
```

Introducing some factory methods can make things much more readable:

```csharp

    public static Result Fail(List<string> errors)
    {
        return new Result(false, errors?.ToArray());
    }

    public static Result Success()
    {
        return new Result(true, null);
    }
```

The sample above now becomes:

```csharp
    public Result InsertBook(string isbn, string title)
    {
        var errors = new List<string>();

        if (string.IsNullOrWhiteSpace(isbn))
            errors.Add("missing_isbn");

        if (string.IsNullOrWhiteSpace(title))
            errors.Add("missing_title");

        if (errors.Any())
            return Result.Fail(errors);

        // Do insert book processing

        return Result.Success();
    }
```

Now, how about the case where a method has a return value? Instead of `Result`, we introduce a class `Result<T>` where `T` is the type of the return value:

```csharp
public class Result<T> : Result
{
    private readonly T _value;

    public T Value
    {
        get
        {
            if (!IsSuccess)
                throw new InvalidOperationException("No value when failed");

            return _value;
        }
    }

    protected internal Result(T value, bool isSuccess, string[] errors)
        : base(isSuccess, errors)
    {
        _value = value;
    }
}
```

The constructor is not exposed publicly. Instead,  we add additional factory methods to the `Result` class:

```csharp
public class Result
{
    public static Result<T> Fail<T>(List<string> errors)
    {
        return new Result<T>(default, false, errors?.ToArray());
    }

    public static Result<T> Success<T>(T value)
    {
        return new Result<T>(value, true, null);
    }
}
```

Here's an example of how to use it:

```csharp
    public Result<Book> GetBook(string isbn)
    {
        if (string.IsNullOrWhiteSpace(isbn))
            Result.Fail<Book>(new List<string> { "missing_isbn" });

        // Get book
        var book = ....

        return book == null
            ? Result.Fail<Book>(new List<string> { "book_not_found" })
            : Result.Success(book);
    }
```

And since only a single error occurs, we can add some additional factory methods for them as well:

```csharp
public class Result
{
    public static Result Fail(string error)
    {
        if (string.IsNullOrWhiteSpace(error))
            throw new InvalidOperationException("Must provide error when failing");

        return new Result(false, new[] { error });
    }

    public static Result<T> Fail<T>(string error)
    {
        if (string.IsNullOrWhiteSpace(error))
            throw new InvalidOperationException("Must provide error when failing");

        return new Result<T>(default, false, new[] { error });
    }
}
```

Which leads us to this:

```csharp
    public Result<Book> GetBook(string isbn)
    {
        if (string.IsNullOrWhiteSpace(isbn))
            return Result.Fail<Book>("missing_isbn");

        // Get book
        var book = ....

        return book == null
            ? Result.Fail<Book>("book_not_found")
            : Result.Success(book);
    }
```

With all the factory methods in place, you can now change the access to the `Result` constructor as protected:

```csharp
protected Result(bool isSuccess, string[] errors)
```

### Handling Exceptions

It is reasonable to think of extending the Result class to handle exceptions:

```csharp
public interface IResult
{
    bool IsSuccess { get; }
    string[] Errors { get; }
    Exception Exception { get; }
}
```

But this would mean that the method would need in include a `try/catch` block to catch and then return the error. I can't dream of all possible uses of this Result class, so it may be something that makes sense for a given context. However, I would urge against such an approach if possible. It is more straightforward to let the exceptions get thrown and expect callers to handle the exceptions as they see fit.

However, using the `Result` class does mean you can avoid throwing exceptions and being more explicit about the way the code works. For example, this might be something you want to do:

```csharp
    try
    {
        _context.Set<Book>().Add(new Book(isbn, title));
        _context.SaveChanges();
    }
    catch (SqlException ex) when (ex.Message.Contains("IX_Book_Isbn"))
    {
        return Result.Fail<Book>("duplicate_isbn_not_allowed");
    }

    return Result.Success();
```

### A Step Towards Functional Programming

One advanced feature you can include with the `Result` class is a way to process the output. You can add a 'Match' method to run actions when the result is returned:

```csharp
public static class ResultExtensions
{
    public static void Match(this Result result, Action success, Action<string[]> fail)
    {
        if (result.IsSuccess)
            success();
        else
            fail(result.Errors);
    }
}

```

Which can turn this:

```csharp
    var result = handler.InsertBook("1234", "title");
    if (result.IsSuccess)
    {
        Console.WriteLine("Success");
    }
    else
    {
        Console.WriteLine("Errors: {0}", string.Join(",", result.Errors));
    }
```

into this:

```csharp
    handler.InsertBook("1234", "title")
        .Match(
            () => Console.WriteLine("Success"),
            errors => Console.WriteLine("Errors: {0}", string.Join(",", errors))
        );
```

Or you can introduce another `Match` function that returns results:

```csharp
    public static T Match<T>(this Result result, Func<T> success, Func<string[], T> fail)
    {
        return result.IsSuccess
            ? success()
            : fail(result.Errors);
    }
```

Which allows you to do this:

```csharp
    var message = handler.InsertBook("1234", "title")
        .Match(
            () => "Success",
            errors => $"Errors: {string.Join(",", errors)}"
        );

    Console.WriteLine(message);
```

There are several other Functional programming patterns that can also be used for such scenarios. So it becomes a very powerful way of returning results from methods.

### Summary

Introducing a `Result` class provides a means to return not only a value from a method, but also contextual information of success/failure of the processing. It can also make it possible to introduce functional programming techniques which can make the code easier to write and understand.


