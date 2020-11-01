---
layout: post
title: 'Column Not Found with EF Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

This week I encountered an odd error while working with EF Core. I hand-crafted a C# class representing an existing table in a legacy database. I added it to my `Context`, just like I had with dozens of other tables. But when I queried the table, EF Core raised an exception. 

<!--more-->

### Missing Column

The exception EF Core raised was:

```
Unhandled Exception: System.Data.SqlClient.SqlException: Invalid column name 'CustomerName'.
```

I checked for typos and found nothing out of the ordinary. I use SQL Server Management Studio to query the table and it returned the values just fine. 

What was going on?

So I decided to try the EF Core scaffolding feature to generate the C# class for me:

```
dotnet ef dbcontext scaffold "Server=localhost;Database=Legacy21;Trusted_Connection=true" Microsoft.EntityFrameworkCore.SqlServer -o .\temp -t [dbo].[CustomerData]
```

The resulting C# class and mapping configuration generated just fine. I copied them into my project and lo and behold it worked. No errors.

### The Mystery

So why did the scaffold-generated classes work and my hand crafted one did not? I assumed I had made some sort of mistake in my version. But when I looked a the the model builder data, I noticed something odd: 

**Hand-Crafted**
```csharp
    entity.Property(e => e.CustomerName)
        .IsRequired()
        .HasColumnName("CustomerName")
        .HasMaxLength(255)
        .IsUnicode(false);
```

**Scaffold-Generated**
```csharp
    entity.Property(e => e.CustomerName)
        .IsRequired()
        .HasColumnName("Customer[ZWSP]Name")
        .HasMaxLength(255)
        .IsUnicode(false);
```

I was using JetBrains Rider and it translated this mystery character to `[ZWSP]`. What was this doing in the column name?

### Zero-Width Space

Turns out this is a thing: 

> The zero-width space (​), abbreviated ZWSP, is a non-printing character used in computerized typesetting to indicate word boundaries to text processing systems when using scripts that do not use explicit spacing, or after characters (such as the slash) that are not followed by a visible space but after which there may nevertheless be a line break.

**Source:** [https://en.wikipedia.org/wiki/Zero-width_space](https://en.wikipedia.org/wiki/Zero-width_space)

This explained everything. This character is not visually rendered in SSMS so the colum name appeared normal. 

The Wikipedia page explained that the character is represented by `U+200B` in Unicode or `&#8203;` in HTML.

So I decided to try it in a query:

```sql
DECLARE @sqlCommand nvarchar(1000)

SET @sqlCommand = 'select [Customer' + NCHAR(8203) + 'Name] from dbo.CustomerData'

EXEC (@sqlCommand)
```

Sure enough, this worked.

And if I looked at the table definition, sure enough it was there all along:

```sql
select COLUMN_NAME, LEN(COLUMN_NAME) AS NAME_LENGTH, REPLACE(COLUMN_NAME, NCHAR(8203), '[ZWSP]') AS [DECODED]
from INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'CustomerData'
```

Using the `NAME_LENGTH` value it was easy to see that the visual column name and the actual column name had different lengths. And the `DECODED` value made it possible to see where these characters were. 

### How Did This Happen?

So this worked out in the end to be easy enough to solve... once I knew what I was looking for. Seeing the hidden character in JetBrains Rider put me on the right path to hunt down the mystery character. I decided to open the files in Visual Studio 2019. Unfortunately, the ZWSP character was not visible in the IDE. Nor was it visible in SSMS. Microsoft may need to do some work in this area to help us poor developers out.

But that begs the question, how did this mystery character get into the database in the first place? I may never know. But I suspect that the table was imported into the database via a utility of some sort, rather than someone using a SQL Script. The legacy database in question has several imported tables, most of them include spaces in the column names. Still not sure how a zero-width space would exist in all this, but I have seen strange things over the years with SQL Server. So anything is possible.

### Summary

If the database says it can't find a column and you don't see any typos in your SQL query, you may just have a hidden character in a column name. It may not be the zero-width space causing the problem, but it couldn't hurt to take a look at the table definition (ie. query the INFORMATION_SCHEMA.COLUMNS table) to find out.
 