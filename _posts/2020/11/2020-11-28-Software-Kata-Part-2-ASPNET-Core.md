---
layout: post
title: 'Software Kata Part 2 - ASP.NET Core'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

As described in Part 1, the concept of [kata][kata] in software development has been around for years. I decided to start a process of regular practice with Angular to retain my skills and to improve them over time.

The same goes for ASP.NET Core, my backend HTTP API development tool of choice. While there are other API frameworks out there, ASP.NET Core is by far the simplest and powerful one I have come across. One can star up a new API project and get it up and running in no time. But I don't create a new API from scratch very often. SO I plan on practicing it as well.

<!--more-->

### The Process

I'm going to start with a simple plan. At regular intervals (yet to be determined) I will practice a set of skills for a given technology. I'll have a pre-defined script of steps I want to complete for each. The script will start small and over time will hopefully grow to include more steps as I become more used to the technology. 

I plan on creating a folder structure on my computer where I can practice. For example, here is where my Kata for Nov 24, 2020 would be for ASP.NET Core:

```powershell
C:\code\kata\2020\11\24\api
```

I plan on focusing this practice on creating an API that my Angular project can call. Ultimately (a 'Part 3' in this series?) will involve establishing proper security between the Angular application and the API. But for now, I'll assume no security is involved.

### The Skills to Master

As with Angular, I want to practice the basics, but I also want to practice organizing the application and getting proper unit tests in place:

- I want to practice creating a new project and build out the start of a simple application. 
- I want to call a database to store and retrieve data. 
- I want unit tests for all the domain logic.

In this context, what will be my domain? I have a set of three readings of the environment (pressure, oxygen and temperature). For this example, I'll assume a single device measures all three values (one at a time). I'll create an in-memory simulator of the device and save the readings into a database table once every 15 seconds. The API will have a GET endpoint where callers can retrieve the latest readings. 

### Summary

Integrating an HTTP API with an Angular app is a very common use case. This kata is meant to practice creating such a service. I should be able to create one from scratch quickly and properly.

[kata]: https://en.wikipedia.org/wiki/Kata_(programming)
