
### In the Beginning

I write a lot of code using C#, specifically, web-based services using ASP.NET Core. When you start creating a new service with ASP.NET Core, you are presented with two 'boilerplate' classes: `Program.cs` and `Startup.cs`. `Program.cs` contains the code that executes when the service applications runs. The `Startup.cs` class contains all the initialization code that configures the service for the environment it is running in.

No unit tests exist for these two classes in the default ASP.NET Core template. Should you write them? I'll admit, I never have. But should I try? I don't think so. Unit tests against code someone else is responsible for should only be written when I need to verify that code behaves the way it should. I've never doubted the boilerplate code behavior and even if I did, in the vast majority of the cases, I would see the unexpected behavior immediately upon running the application. 

So this is my starting point. Two boilerplate classes. What comes next is up to me. I may be interested in a top-down approach where I am writing code to model an HTTP endpoint or controller. Or I maybe interested in writing in a bottom-up approach focusing on the domain logic. In either case (or any other you may want to explore), practicing TDD means you have to write a test to begin. Let's start there.
