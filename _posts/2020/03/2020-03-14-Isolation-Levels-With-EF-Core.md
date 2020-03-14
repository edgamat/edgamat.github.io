---
layout: post
title: 'Isolation Levels with EF Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
When using Entity Framework Core, transactions are a fact of life. When using `TransactionScope`
to explicitly manage business transactions there are some gotchas you should be aware of.

<!--more-->

## What are Isolation Levels?

A lot of the time you don't need to concern your self with isolation levels using Entity Framework. However, when
accessing an existing database of significant size, or where performance is critical, they can be something you may need
to investigate when diagnosing poor performance or worse, deadlocks. 

Here is a great description of isolation levels:

> Transactions specify an isolation level that defines the degree to which one transaction must be isolated from resource or data modifications made by other transactions. Isolation levels are described in terms of which concurrency side effects, such as dirty reads or phantom reads, are allowed.
> 
> Transaction isolation levels control the following:
> 
> - Whether locks are taken when data is read, and what type of locks are requested.
> 
> - How long the read locks are held.
> 
> - Whether a read operation referencing rows modified by another transaction:
> 
>     - Block until the exclusive lock on the row is freed.
> 
>     - Retrieve the committed version of the row that existed at the time the statement or transaction started.
> 
>     - Read the uncommitted data modification.

Source: "Understanding Isolation Levels" [https://docs.microsoft.com/en-us/sql/connect/jdbc/understanding-isolation-levels?view=sql-server-ver15][understanding]

## Our Problem

In a recent SQL Server based project, our use of Entity Framework involved a few complex business transactions that included multiple reads to the database, followed by stored procedure calls and more than one call to `SaveChangesAsync`. We decided to wrap all this activity with a database transaction to ensure data consistency. Since the project used EF Core 2.1, we decided to use the [built-in support of TransactionScope objects][ef-core-transactions].

```csharp
using (var scope = new TransactionScope())
{
    // do work here

    scope.Complete();
}
```

The transaction is not committed when the call to `Complete()` is made, but rather when the object is disposed. If no call to `Complete()` is made, then the transaction will roll back during the dispose.

When testing locally, no issues with this approach occurred. However, in the shared development environment, we saw deadlocks occur frequently. Especially when running the system under a high load. And we didn't see any issues of this kind prior to using the explicit transactions.

The first assumption was the duration of the transactions was long enough to cause problems. So we started looking at the queries we wrote using `Linq` and the stored procedures to find possible improvements.

When looking closer at some of the stored procedures being called, we noticed the extensive use of the `NOLOCK` hint:

```sql
SELECT * FROM [dbo].[Customer] WITH (NOLOCK) WHERE Id = @Id
```

When we asked one of the DBAs about this, they indicated it was used to reduce locks on some of the tables due to the high volume of operations being done. But since most data was write-once, it didn't have any serious side effects (like dirty reads).

The `NOLOCK` hint allows the queries to ignore any locks. This improves performance because they won't be blocked by other processes. However, it can mean that it may read data that is not committed yet. If that data is rolled back, it is considered a dirty read and is most undesirable. 

With this new knowledge in hand, we set out to try and determine what isolation level was being used, and how to change it if necessary.

## Default Behavior

When calling `SaveChanges` or `SaveChangesAsync` Entity Framework performs all changes using a single transaction. This ensures data consistency: everything is saved or nothing is. No surprises here, I would assume. 

These transactions use the SQL Server defaults. Which for our environment (EF Core 2.1 and SQL Server 2014) was `READ COMMITTED`.

`READ COMMITTED` is a pretty safe place to be. It uses minimal locks and can share locks with other processes. But it does
so without allowing dirty reads to occur.

Using SQL Server Profiler (a must-have when using Entity Framework) this was the confirmed behavior when no `TransactionScope` was in use.

When looking at the statements being executed with `SaveChanges` from within a `TransactionScope`, the isolation levels had changed. The isolation level was now set to `Serializable`, which is the same as using the `HOLDLOCK` hint. It is the most restrictive isolation level partially because it won't let other processes insert new rows with key values that would fall in the range of keys read by any statements in the current transaction until the current transaction completes (a.k.a. range locks).

This was a surprise to us so we looked for a way to control this behavior. 

## Our Solution

We ended up changing all uses of `TransactionScope` to the following:


```csharp
using (var scope = new TransactionScope(
    TransactionScopeOption.Required,
    new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted }))
{
    // do work here

    scope.Complete();
}
```

Again using SQL Server profiler we confirmed that all queries we done using `READ COMMITTED`. The dead-lock issues and slow performance no longer occurred in the shared development environment. And since moving the code into testing and production, no reports of dead locks have been made.

## Points of Interest

### Connection Pooling

When using SQL Server Profiler, something interesting was observed when the use of `TransactionScope` objects resulted in Serializable isolation levels. Even queries made without the use of `TransactionScope` were using Serializable isolation levels.

Since EF Core uses the default connection pooling, the connection with the Serializable isolation level is reused for other commands, even reading data via `Linq` queries. The application uses Serilog to record log entries in SQL Server. Even its commands to write to the log table ended up using the connection with the Serializable isolation level.

I found a Stack Overflow article that discusses it: [https://stackoverflow.com/questions/3759897/how-does-sqlconnection-manage-isolationlevel][so-article]

Once a connection is set to have a specific isolation level, it will remain so until it is removed from the connection pool.  

Knowing this was the result of using the default settings of the `TransactionScope` I was drawn back to the docs page on the Microsoft website for this EF Core feature. And sure enough, the examples on the page were NOT using the defaults, but rather explicitly setting the isolation level to READ COMMITTED. It's like they knew....

### Async/Await

Don't forget to use the explicit `TransactionScopeAsyncFlowOption` parameter to handle asynchronous calls when using TransactionScope:

```csharp
using (var scope = new TransactionScope(
    TransactionScopeOption.Required,
    new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted },
    TransactionScopeAsyncFlowOption.Enabled))
{
    // do work here

    await dbContext.SaveChangesAsync(token);

    scope.Complete();
}
```

 


[understanding]: https://docs.microsoft.com/en-us/sql/connect/jdbc/understanding-isolation-levels?view=sql-server-ver15
[ef-core-transactions]: https://docs.microsoft.com/en-us/ef/core/saving/transactions#using-systemtransactions
[so-article]: https://stackoverflow.com/questions/3759897/how-does-sqlconnection-manage-isolationlevel
