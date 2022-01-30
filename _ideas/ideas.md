"Persistance without insight leads to the same outcome"

DONE! The Books I Read in 2021

DONE! How to organize service registration with the Microsoft Dependency Injection Container


Add eslint and prettier to a NodeJS project (with TypeScript)
https://dev.to/caelinsutch/setting-up-a-typescript-nodejs-application-with-prettier-and-eslint-53jc#:~:text=As%20in%20the%20name%2C%20Prettier%20makes%20you%20code,to%20set%20the%20project%20up%20as%20node%2Fnpm%20project.
https://prettier.io/docs/en/install.html
https://github.com/prettier/eslint-config-prettier
https://eslint.org/docs/user-guide/configuring/configuration-files#configuration-file-formats

DONE! Decorator Pattern with the Microsoft Dependency Injection Container

Captive Dependencies

https://levelup.gitconnected.com/top-misconceptions-about-dependency-injection-in-asp-net-core-c6a7afd14eb4
https://blog.ploeh.dk/2014/06/02/captive-dependency/
Captive dependencies is a kind of dependency injection anti-pattern that can lead to bugs in your code.
All you have to do to get a captive dependency problem is to inject a class with a shorter lifetime into a class with a longer lifetime. For example, inject a scoped service into a singleton. In this case, a scoped class will not be created and disposed per each web request. A scoped class will have the same lifetime as a singleton, and actually will become a singleton.

Dependency Injection Container Always Disposes Objects

```C#
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(new Service());
}
```

DONE! Unit Testing Dependency Injection Container Registrations

How to use Dependency Injection with a Console Application

Using Autofac with an .NET 6 Application


Will the Real CURL please stand up?

Message Types in a Micro Services Architecture
    - Commands
    - Events
    - Documents

Reusing objects in unit tests

Using BCP to ETL data

Using WSL 

Why are Coding Standards Important?

Configuring Angular Applications




Using ROBOCOPY to transfer files

Using PowerShell to Read/Write Files

Using Posh Git

1Password after one year

Dependency Injection with .NET

Writing Background Workers in .NET

Diagnosing Memory Leaks with JetBrains dotMemory


Hammer and Nails
- For every job there is the perfect tool
- Finding reasons to use your favorite tool




Skills

Pattern Recognition
Situational Awareness
Big Picture
Kindness
Communication
Do the Work


Flight Rules

Flight Rules are the hard-earned body of knowledge recorded in manuals that list, step by step, what to do it X occurs and why. Essentially, they are extremely detailed, scenario-specific standard operating procedures.

Flight Rules protest against the temptation to take risks, which is strongest when momentum has been building to meet a deadline.

Standard Operating Procedures

101) When making a change to the code
102) When fixing a bug
103) When creating a Pull Request
104) When adding a feature not ready for production
105) When adding an endpoint
106) When adding a message consumer
107) When adding a message producer
108) When adding a log entry
109) When handling an exception
110) When adding an application setting
111) When modifying an application setting
112) When adding a health check
113) When adding a new table/view to an EF Core context (not owned by the project)
114) When adding a call to a stored procedure (not owned by the project)

201) When a build fails in DEV
202) When a deployment fails in DEV
203) When a deployment fails in TEST
204) When a nightly integration test fails
205) When a nightly security scan fails
206) When a deployment fails in PROD


JSON Encoding Dates (to include timezone info when serializing) 

Using new HealthChecks in ASPNET

Legacy EF Core - triggers

Using default values with EF Core

Windows Event Logs with Serilog

Two dashes with Windows Event Log Sources


https://dev.to/draft/the-3-questions-you-should-ask-before-starting-a-technical-blog-b8h

### What kind of content will you write?
“If you think something is clever and sophisticated, beware - it is probably self-indulgence.” - Donald A. Norman, The Design of Everyday Things

Your content should be unique, but it shouldn’t surprise your audience. Most successful blogs choose 2-3 standard "formats" and publish them consistently. Here are the most common formats I see on software engineering blogs:

+ **Tutorials** - These step-by-step walkthroughs show readers how to build something or illustrate a specific use-case for your product. They provide a lot of value to engineers who are using your tool or related technologies.

+ **Roundups** - Listacles get a bad wrap, but high-quality curation is still valuable when done right. Roundups bring together a wealth of information to help your readers make decisions.

+ **Case Studies** - Like tutorials, case studies can illustrate a use-case for your product, but they tend to be more focused on the outcomes than the process. Case studies can also come off as a bit “salesy” if you're not careful.

+ **Features** - Borrowing the word from journalism, a “feature” dives deep into something of interest to your audience. For example, you might introduce a new feature, highlight a team member, or do a technical teardown of part of your codebase.

+ **Q&As** - Interviews with team members, customers, and influencers in the industry can be a great way to focus your blog’s lense outwards and help you piggyback off your interviewee's audience.

+ **Comparisons** - Comparisons help readers decide between competing options. A good comparison post will offer an objective conclusion for which option is best under which circumstance.

+ **Opinion Pieces** - Personal experiences and observations help establish you as a thought leader. These pieces are valuable when done well, but require a bit of name recognition to pull them off.
