# How To Avoid Automatic Evaluation with EF Core

https://saebamini.com/the-dangerous-ef-core-feature-automatic-client-evaluation/

When using EF Core, the default configuration will not warn you (with a runtime error) if you 
are using a Linq query that EF Core cannot evaluate. This will result in the query being
run in a manner that might not be as expected. For example:

var query = context.Set<Blog>().Where(x => x.Url.Equals("google.com/", StringComparison.OrdinalIgnoreCase));
var results = query.ToList();

The overloaded Equals method with the StringComparison argument is not something EF Core understands. As
a result, the generated query has no WHERE clause and it retrieves all the records from the Blog table, until
a match is found. In other words, if you expected it to generate this:

```SQl
SELECT BlogId, Url
FROM Blog
WHERE LOWER(Url) = LOWER('google.com/')
```

Or something similar, you are going to be disappointed. It actual generates this:

```SQL
SELECT BlogId, Url
FROM Blog
```

To avoid this behavior, you can configure EF Core to throw a runtime error if it encounters something it cannot
covert into a SQL statement.

When configuring the options, set the following:

```CSharp
optionsBuilder
    .UseLoggerFactory(MyLoggerFactory)
    .ConfigureWarnings(warnings => warnings.Throw(RelationalEventId.QueryClientEvaluationWarning))
    .UseSqlServer("Server=(localdb)\\mssqllocaldb;Database=EFCoreTests.NewDb;Trusted_Connection=True;ConnectRetryCount=0");
```

Then you will know when your EF-generated queries is not quite what you might have expected.