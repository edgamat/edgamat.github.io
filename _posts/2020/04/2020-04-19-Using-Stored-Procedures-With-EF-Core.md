---
layout: post
title: 'Using Stored Procedures with EF Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Sometimes, especially when using a legacy database, it is necessary to call stored procedures. Here are some of the ways possible with EF Core using SQL Server.

<!--more-->

## A Basic Command

The stored procedure in SQL Server has four different parameter types:

- Input
- Output
- Input/Output
- Return Value

The interface EF Core provides to call stored procedures is very similar to the mechanism used by the underlying ADO SqlClient interfaces. You define an array of parameters, execute the command and inspect the parameters.

### Sample Parameters

Here is a sample stored procedure with two input parameters and an output parameter

```csharp
const string sql = "[dbo].[SampleProcedure] @FirstParam, @SecondParam, @Message OUTPUT";
```

Here is how to call this from C# using EF Core:

```csharp
var message = new SqlParameter
{
    DbType = DbType.String,
    Direction = ParameterDirection.Output,
    ParameterName = "@Message",
    Size = 255
};

var parameters = new object[]
{
    new SqlParameter("@FirstParam", DbType.Int32) { Value = 123 },
    new SqlParameter("@SecondParam", DbType.Int32) { Value = 456 },
    message
};

var result = await _context.ExecuteSqlCommandAsync(sql, parameters, token);

var messageValue = message.Value.ToString();
```

The `return` value is the number of rows affected by the stored procedure. This value may be zero if the stored procedure includes a `SET NOCOUNT ON` directive. So be careful when using the return value.

## Transactions

When combining EF Core `SaveChanges/SaveChangesAsync` with store procedure calls, it is necessary to use an explicit transaction in order to commit/rollback all the changes. There are two mechanisms to create a transaction. 

### IDbContextTransaction (Default Behavior)

[The EF Core docs online][saving-trans] include a great description of using transactions. The default behavior with EF Core is to create a transaction and then commit it within a using statement:

```csharp
using (var trans = await _context.Database.BeginTransactionAsync(IsolationLevel.ReadCommitted, token))
{
    // Do work
    _context.SaveChangesAsync(token);

    // Call stored procedure
    var _ = await _context.ExecuteSqlCommandAsync(sql, parameters, token);
    
    trans.Commit();
}
```

NOTE: Always use an explicit IsolationLevel when beginning a transaction. The default IsolationLevel is difficult to remember and you'll save a lot of time debugging issues (and there will be issues) when this is set explicitly.

### `TransactionScope`

Starting with EF 2.1, `TransactionScope` can be used instead of `IDbContextTransaction`. The pattern is virtually the same:

```csharp
using (var scope = new TransactionScope(
    TransactionScopeOption.Required,
    new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted },
    TransactionScopeAsyncFlowOption.Enabled
    ))
{
    // Do work
    _context.SaveChangesAsync(token);

    // Call stored procedure
    var _ = await _context.ExecuteSqlCommandAsync(sql, parameters, token);
    
    scope.Complete();
}
```

Again, be explicit with the `Isolationlevel` being used. The `TransactionScopeAsyncFlowOption` allows the code to use the `async/await` syntax and work as expected.

### Use `Scoped` Lifetime

When using `TransactionScope` it is critical that the DbContext is injected into the DI pipeline using a Scoped lifetime. If not, EF Core will raise a runtime exception relating to unsupported distributed transactions.

I have a suspicion that it is possible to have a Transient lifetime registered DbContext but would require a Scoped Transaction to be registered in the DI pipeline. Not sure there's a benefit to that approach, but there may well be.

## Summary

Calling stored procedures in EF Core is similar to how it is done using the underlying SqlClient ADO objects. Be explicit with the IsolationLevel and use transactions when mixing SaveChanges() and stored procedures. 

[saving-trans]: https://docs.microsoft.com/en-us/ef/core/saving/transactions


