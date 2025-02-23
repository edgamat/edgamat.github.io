---
layout: post
title: 'Scripting SQL Server Users'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Do you know what permissions a user has in SQL Server? Generating scripts to create users
and grant permissions can save you a lot of trial and error.

<!--more-->

### Automate Everything

For .NET applications that interact with SQL Server, it is necessary to grant the account
running the application access to SQL Server. You can grant the account too many permissions or
too many. It takes care and analysis of the application needs to get the permissions just right.
Once you have the permissions dialed in, it is important to record them (somewhere) so you don't
have to go through the effort of figuring it out all over again in the future.

The best choice for this is a script that you can give to an administrator and have them apply the
necessary roles/grants to the user for a given database. Ideally this script should be under source
control so you can keep track of it (don't loose it) and see how changes are made to the permissions
over time.

But when you are first setting out on this journey to create the scripts, you may be in the awkward
position of not knowing what the permissions are. Thankfully there are some tools you can use to
generate the scripts based on the current permissions.

### SQL Server Management Objects

There is a Nuget package you an use to query the database metadata and create scripts. It is called
the SQL Server Management Objects (SMO) package: `Microsoft.SqlServer.SqlManagementObjects`

This package allows you to programmatically interact with a SQL Server instance. I won't go into all
the capabilities, but it can do a lot. For our needs, we want to use it to:

- Create DROP and CREATE statements existing Database User
- Enumerate all the database roles a Database User belongs to
- Enumerate all the database permissions a Database User has
- Enumerate all the schema permissions a Database User has

To get started, create a Console application and add the following 2 packages:

```bash
dotnet add package Microsoft.Data.SqlClient
dotnet add package Microsoft.SqlServer.SqlManagementObjects
```

The programming model revolves around a Server and a Database object:

```csharp
using Microsoft.Data.SqlClient;
using Microsoft.SqlServer.Management.Common;
using Microsoft.SqlServer.Management.Smo;

using SqlScripts;

var loginName = "MyLoginName";
var databaseName = "MyDatabaseName";

using var conn = new SqlConnection($"Server=localhost;Database={databaseName};Integrated Security=True;TrustServerCertificate=True");
var srv = new Server(new ServerConnection(conn));
var db = srv.Databases[conn.Database];

var userToScript = db.Users[loginName];
```

### Scripting Objects

In the SQL Server Management Studio (SSMS), there is a feature to script objects in a database. It
has numerous options to control how the resulting scripts are generated. THe SMO programming model
includes the same scripting capabilities. For example, here is how you can script the DROP and
CREATE statements for a user:

```csharp
var scripter = new Scripter(srv);

scripter.Options.ScriptDrops = true;
scripter.Options.IncludeIfNotExists = true;

var dropScript = scripter.Script(new Urn[] { userToScript.Urn }).Cast<string>().ToList();
dropScript.ForEach(Console.WriteLine);

scripter.Options.ScriptDrops = false;
scripter.Options.ScriptForCreateDrop = true;
scripter.Options.IncludeIfNotExists = false;

var createScript = scripter.Script(new Urn[] { userToScript.Urn }).Cast<string>().ToList();
createScript.ForEach(Console.WriteLine);
```

### Enumerating Permissions

The SSMS UI has menus, windows and dialogs for viewing the permissions for database objects. But
they are very confusing at times and it feels like it is impossible to see them all in once place.

The SMO programming model allows you to enumerate all the permissions and list them. For example,
here are the permissions assigned to a User in the current database:

```csharp
foreach (DatabasePermissionInfo perm in db.EnumDatabasePermissions())
{
    if (perm.Grantee == userToScript.Name)
    {
        Console.WriteLine($"GRANT {perm.PermissionType} TO [{userToScript.Name}];");
    }
}
```

And here are all the schema permissions:

```csharp
var schemaPermissions = db.Schemas.Cast<Schema>()
    .SelectMany(s => s.EnumObjectPermissions())
    .Where(p => p.Grantee == userToScript.Name)
    .Select(p => $"GRANT {p.PermissionType} ON SCHEMA::{p.ObjectName} TO [{userToScript.Name}];")
    .ToList();
schemaPermissions.ForEach(Console.WriteLine);
```

And lastly you can enumerate all the role the User is a member of:

```csharp
var roleMembers = db.Roles.Cast<DatabaseRole>()
    .Where(r => r.EnumMembers().Contains(userToScript.Name))
    .Select(m => $"EXEC sp_addrolemember '{m.Parent.Name}', '{m.Name}';")
    .ToList();
roleMembers.ForEach(Console.WriteLine);
```

### Summary

Using the SMO programming model, you can generate a script to recreate an existing user. It can then
be placed in source control and used for future reference so you don't have to relearn how you set
up the user the first time.
