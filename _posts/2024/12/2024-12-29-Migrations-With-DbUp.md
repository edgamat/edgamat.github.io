---
layout: post
title: 'Creating Migration Scripts for DbUp in .NET'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

DbUp is a great tool for automatically applying database changes when you deploy you code. But it can be difficult to review the scripts when changing stored procedures, functions or views. Let's explore one way of improving this situation using scripted migrations.

<!--more-->

### The Problem

We have a legacy .NET application whose database schema we don't control. We can add custom tables, views, stored procedures and functions, but in doing so we open ourselves to managing these objects on our own. Until now, we have not had a means of including these changes with the code as it is deployed. As a result, the database changes have been performed manually. This creates a higher risk of forgetting something during a deployment, or not knowing when or why changes have been made.

One popular solution is to deploy the database changes when you deploy your code. This can be done with EF Core or DbUp without too much effort. But let's say for example we have a set of stored procedures we want to maintain using these migrations. Each migration script would include a version of the updated stored procedure. The migration gets applied when the code is deployed. All is good in the world.

But this is a huge pain point for the code review process. If we have a code review process that looks at the changes before they are deployed, what do we give the reviewers? The migration script? Well, it only contains the version of the procedure being deployed. It doesn't provide reviewers with the changes being made. This is the problem we are trying to solve. We want the convenience and simplicity of automated migrations being automatically applied when the code deploys. But we also want reviewers to be able to see the differences being made so they can understand what is being done.

### Proposed Solution

I expect that there are a number of ways to solve this problem. But the solution I want to explore is the idea of auto-generating the migration script using the SQL scripts. Let's say you have a folder that contains all the stored procedures. This folder is under source control along with the rest of the .NET code. When a code review is requested, you can include these files to see the changes being made.

We can then use these files to generate the migration script. Developers don't have to create the migration script manually, they will use a command line tool to generate it, similar to how EF Core code-first migrations work.

The trick is to find a way to know which of the scripts have changed. For that, I propose we use a bit of inspiration from EF Core, namely its snapshot file. EF Core maintains a snapshot file that contains the schema definition based on all the configured model classes. If a model class changes (or is removed or added), EF Core will generate the necessary migration scripts and update the snapshot file.

I propose a similar approach with the stored procedures. We create a snapshot file (snapshot.txt) that contains a list of all the script files it manages. Along with the relative path to the file, we also include an MD5 hash of the file contents.

```text
./db-scripts/01-tables/dbo.Customers.Create.sql F7C76D3C8949523CCC5CAD6F2B0DBBE9
./db-scripts/01-tables/dbo.Orders.Create.sql    E2317E9C32E0C70974971B70117C0BD5
./db-scripts/03-procedures/dbo.DeleteCustomer.sql   DCF586B0B9A897EB9241DC921459E991
./db-scripts/03-procedures/dbo.GetCustomerDetails.sql   44DB6DE7552C9494D6916E228523A606
./db-scripts/03-procedures/dbo.GetOrderDetails.sql  FB29D5FA698A2A3A0422EAB0306C4BC9
```

If a new script file is added, we would add a new line to the snapshot file. If the MD5 hash of an existing file changes, we update the hash in the snapshot file. If a script file is deleted, we remove the line from the snapshot file. All these changes can be tracked, and used to generate the migration script. For files that are added or updated, we include the contents of the script file in the migration script. For files that are removed, we add a "DROP" statement that will remove it from the database.

### Implementation with PowerShell

There are a number of different ways to implement this command line tool. I have chosen to use a PowerShell script, because writing PowerShell scripts is something I need more practice with. But this could easily be written as a .NET console application.

Here is a GitHub gist that has the 2 PowerShell scripts:

[https://gist.github.com/edgamat/2c5125e8f7c434eddf382f6781b1a2aa](https://gist.github.com/edgamat/2c5125e8f7c434eddf382f6781b1a2aa)

The AddMigration.ps1 script is used to scan a folder called `db-scripts` and get a list of all the SQL Scripts. It compares this list to the one in the `snapshot.txt` file, found in the `migrations` folder. It returns the list of changes (added/updated/deleted) and updates the `snapshot.txt` file accordingly.

Then it uses the list of changes to create a new migration script, named:

`{yyyyMMdd_HHmmss}_{MigrationName}.sql`

The `MigrationName` is passed into the script:

```powershell
.\AddMigrations.ps1 "MyMigration"
```

One issue that comes from this approach is adding scripts to the migration script in the correct order. For example, you would want to create tables before creating stored procedures that reference these tables.

To deal with this issue, the scripts are divided into sub folders with names that imply the correct order:

- **01-tables**: A subfolder containing all the CREATE/ALTER TABLE scripts
- **02-views**: A subfolder containing all the DROP/CREATE VIEW scripts
- **03-procedures**: A subfolder containing all the DROP/CREATE PROCEDURE scripts
- **04-functions**: A subfolder containing all the DROP/CREATE FUNCTION scripts
- **05-data**: A subfolder containing all INSERT/UPDATE/DELETE scripts

You may find that in your situation, the order of these might need to change (perhaps functions should be done before stored procedures). In addition, you may want to include a separate folder for triggers.

In the 01-tables folder, there is always a script used to create the table and then additional scripts for each alteration. To ensure these are executed in the correct sequence, we recommend the following naming convention:

```powershell
{schema}.{TableName}.00000.sql # CREATE script
{schema}.{TableName}.00001.sql # First ALTER script
{schema}.{TableName}.00002.sql # Second ALTER script
...
```

In the `02-views`, `03-procedures`, and `04-functions` folders, you will create your DROP/CREATE scripts for the respective objects. Each script must be idempotent (running it multiple times will not cause an error). For example:

```sql
if (object_id('dbo.DeleteCustomer') is not null)
BEGIN
  DROP PROCEDURE dbo.DeleteCustomer
END
GO

CREATE PROCEDURE dbo.DeleteCustomer
  @CustomerID UNIQUEIDENTIFIER
AS
BEGIN
  UPDATE Customers
  SET IsDeleted = 1
  WHERE ID = @CustomerID
    AND IsDeleted = 0
END
GO
```

In the `05-data` folder, you will create any INSERT/UPDATE/DELETE scripts for manipulating data. The recommended naming convention is:

```sh
00001.{FirstScriptName}.sql
00002.{SecondScriptName}.sql
00003.{ThirdScriptName}.sql
...
```

### Incorporating with DbUp

Once the migration script is created, we need to make it available for DbUp to use. In my scenario, DbUp will look for any embedded files in the .NET assembly where the scripts are found. So the migration file must be an embedded resource file. The `AddMigrations.ps1` script will update the C# project to add the proper elements to make the migration file an embedded resource:

```xml
  <ItemGroup Label="Migrations">
    <EmbeddedResource Include="migrations\20241223_144940_NewMigration.sql" />
  </ItemGroup>
```

In the project, you can include a helper class to execute the migration scripts:

```csharp
public static class DataMigrations
{
    public static DatabaseUpgradeResult ExecuteMigrations(string connectionString)
    {
        var upgrader =
            DeployChanges.To
                .SqlDatabase(connectionString)
                .JournalToSqlTable("dbo", "__SchemaVersions")
                .WithScriptsEmbeddedInAssembly(typeof(DataMigrations).Assembly)
                .LogToConsole()
                .Build();

        var result = upgrader.PerformUpgrade();

        return result;
    }
}
```

Lastly, we run the DbUp migrations to apply the migration scripts:

```csharp
var connectionString = "..."; // get from the `appsettings.json` file

var result = DataMigrations.ExecuteMigrations(connectionString);

if (!result.Successful)
{
    Console.ForegroundColor = ConsoleColor.Red;
    Console.WriteLine(result.Error);
    Console.ResetColor();
    return -1;
}

Console.ForegroundColor = ConsoleColor.Green;
Console.WriteLine("Success!");
Console.ResetColor();
return 0;
```

### Summary

It can be challenging to manage database schema changes in a legacy .NET application. We can use tools like DbUp to automate the deployment of changes, but it can make reviewing the changes difficult (and time consuming). But auto-generating migration scripts from SQL files stored in source control can be a possible solution. By using a snapshot file to detect changes and organizing scripts into sub folders, we can automate the creation of migration scripts and streamline the deployment process.
