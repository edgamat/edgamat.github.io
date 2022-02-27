
In most applications you will encounter cases where one component depends on another component and it depends on yet another component. This is called an "object graph" and represents all the components and their relationships.









[Dependency Injection Principles, Practices, and Patterns][dibook2] by Mark Seemann.



[dibook2]: https://www.manning.com/books/dependency-injection-principles-practices-patterns




Take for example the Entity Framework DbContext class. Applications that implement one of these classes to access a database typically register them using as Scoped lifestyle. That means that a separate DbContext is created for each web request processed by the ASP.NET pipeline. This is good, because the DbContext class is not thread-safe. 

If your controller depends on classes that access the database, each of these classes will have the same instance of the DbContext class injected into their constructors. This can lead to confusion when the classes are trying to access the database. 


Interception

Interception is the ability to intercept calls between two collaborating
components in such a way that you can enrich or change the behavior of
the Dependency without the need to change the two collaborators themselves.

