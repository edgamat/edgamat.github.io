---
layout: post
title: 'Verifying Structure with EF Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Sometimes when using EF Core with an existing database, the underlying database structure can change without you knowing. Here is how you can verify the structure of the database matches the code.

<!--more-->

### The Problem

It is possible to use EF Core to query and manipulate data in a database that your application does not manage. For example, an existing database that is managed by a different development team. They make changes the the database according to their workflow and coordinate with other teams when the structure changes.

Instead of using code-first migrations, you and your team are made aware of the changes to the database and you adjust your EF Core model manually. While not perfect, it can work rather efficiently.

One of the benefits of the code-first approach using migrations is the code and database structure are always kept in sync. As long as the migrations are run,  you never have to worry that a runtime query might fail due to a difference in the data structure. It would be nice to have something similar when you don't have control over when the database structure might change.

### The Solution

One solution would be to query the EF Model and determine all the corresponding tables and columns it is aware of in the database. You could then compare that to the metadata from the database and report any differences at application startup. With SQL Server one can obtain this metadata using the `INFORMATION_SCHEMA` tables and views. For example, here is how one might get the list of columns for a table:

```sql
SELECT *
FROM INFORMATION_SCHEMA.COLUMNS
WHERE SCHEMA = 'dbo'
AND TABLE_NAME = 'Customer'
```

And for the EF model side of things, you can get the list of tables and columns via the `DbContext`:

```csharp
var entities = context.Model.GetEntityTypes().ToList();

foreach (var entity in entities)
{
    var tableInfo = entity.Relational();
    // tableInfo.Schema
    // tableInfo.TableName
    
    var properties = entity.GetProperties().ToList();
    foreach (var property in properties)
    {
        var columnInfo = property.Relational();
        // columnInfo.ColumnName
        // columnInfo.ColumnType  e.g.  decimal(12, 8)
        // columnInfo.IsFixedLength
        //property.IsNullable
    }
}
```

The last thing to do is match column types. SQL Server's metadata separates the data type into distinct fields:  

- DataType
- DataLength
- DataPrecision
- DataScale

Here is the function to combine them to match what EF Core has:

```csharp
public string GetSqlColumnType()

    var string[] keywordTypes =
    {
        "int",
        "tinyint",
        "smallint",
        "bigint",
        "byte",
        "money",
        "text",
        "float",
        "real"
    };

    if (DataLength == null && DataPrecision == null)
        return DataType;

    if (keywordTypes.Contains(DataType))
        return DataType;

    if (DataLength != null)
    {
        if (DataLength == int.MaxValue)
            return DataType;

        if (DataLength == -1)
            return $"{DataType}(max)";

        return $"{DataType}({DataLength})";
    }

    if (DataPrecision != null)
    {
        return $"{DataType}({DataPrecision}, {DataScale})";
    }

    return DataType;
}    
```

### Summary

If you need to compare your model to an actual database, it is possible with a little bit of work. Grab the metadata from the database representing the actual model. Grab the 'Relational` data from the EF Core DbContext and compare the two. Any differences are things you need to take a look at.
