---
layout: post
title: 'Using Shared Databases with EF Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Sometimes it isn't possible to have a nice clean database to work from with EF Core. Using EF Core with
an existing database is well understood. However, there are a few gotchas.

<!--more-->

## Background

One project I have been fortunate to work on uses EF Core with an existing database. This database is 
relatively large (1 TB) and contains over 400 tables. The database is the primary data store for a line-of-business
application with hundreds of active users. 

We are writing an application to provide an HTTP API to read and write data in this database. Using the EF Core scaffolding features, we have created a database-first set of classes and a traditional EF Core DbContext for accessing the tables in this database.

## The Problem

Our HTTP API Application does not own any of the tables, and more importantly, the data is being manipulated by other 
applications. And to make matters even more challenging, several of the tables in the database have triggers that
manipulate the database on insert and update.

These conditions results in EF Core being unaware of side-effects to the data. For example, one table includes a trigger
to fill in certain fields that have NULL values. These updates don't appear in the DbContext after `SaveChanges` is called (`SaveChanges` can mean `SaveChanges` or `SaveChangesAsync` depending on your codebase)

In addition, even without triggers, there are several existing stored procedures that must be used to maintain data integrity
and apply business rules. After these procedures are called, there is no guarantee the in-memory object is still representing the same data in the database.

## Attempts to Reduce Side Effects

The following are some patterns and techniques we found to reduce side-effects in such an environment.

### (Carefully) Use Transactions

When using EF Core `SaveChanges` with stored procedures, you typically will need to ensure all the modifications made by EF Core and the stored procedure are done in a single database transaction. Use `TransactionScope` to implement an explicit transaction
for all operations that represent the unit of work.

When using `TransactionScope`, please be aware of its default isolation levels. Use the provided overloaded constructors to explicitly set the isolation that best suits your needs. Please read here for more information about [Isolation Levels with EF Core]({% post_url /2020/03/2020-03-14-Isolation-Levels-With-EF-Core %}).

### Call Stored Procedures after `SaveChanges`

The work performed by stored procedures may rely on the data currently tracked by the DbContext object. You must save all
these changes to the database before calling store procedures. 

### Use `ReloadAsync()` to Ensure Data Is still 'Unchanged'

It is sometimes necessary to perform additional processing logic after calling a stored procedure. However, the entities in the DbContext may have been modified by the stored procedure. Alternatively, a table may have a trigger that modifies the data which
results in the same stale data being tracked in the DbContext.

To ensure the DbContext still contains an accurate representation of the data, call the `ReloadAsync()` method to reload entities from the database. Often, this step is not necessary as the store procedure is likely the last operation called in a business transaction. However, when trigger modify the data, it is often wise to reload the data to ensure transactional consistency.

```csharp
await _dbContext.Entry(customer).ReloadAsync(token);
```

NOTE: It requires analysis of each stored procedure and trigger to determine exactly what side effects are occurring when they execute. This is a critical step in the process and must not be forgotten to be done.

## Summary

When sharing a database between an EF Core application and an exiting application, you must be careful to understand how the existing application modifies the data. Use `TransactionScope` when coordinating work between EF Core and stored procedures.
When using existing stored procedures, ensure `SaveChanges` is called prior to executing the procedure. And when necessary, call the `ReloadAsync` method to ensure the DbContext has all the changes after a stopred procedure or trigger modifies the data.
