---
layout: post
title: 'Software Kata Part 3 - Progress Update'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

As described in Parts 1 and 2, the concept of [kata][kata] in software development has been around for years. I decided to start a process of regular practice with Angular and ASP.NET Core to retain my skills and to improve them over time. Here is my progress update on what I learned from completing only two sessions.

<!--more-->

### The Process, So Far

I decided to start by creating a new application with Angular and ASPNET Core once a week. I felt I could commit time each week to practice. I created the Angular project first and then then ASPNET project. Overall it took about 4 hours to get a working application. 

I discovered that it was very difficult to create new applications 'from scratch'. I made mistakes and had to start over a couple times before getting things right. I realized this was the reason I wanted to start this process in the first place. It is difficult to remember how to do tasks if you don't practice them. No surprise really, as with most things, practice helps.

I had a second 'session' this week and I found that I was trying to use the application from last week as a guide. I got half way through it before I realized I was 'cheating' myself by using the code from last week as a guide. What I wanted to practice was the process of creating a new application, not copying an existing design. But was this really cheating?

### The 'Power' of Code Reuse

When using the code from my previous 'kata' session as a guide, I found I could avoid the mistakes I made the first time around and I was able to get the tasks done more quickly. I didn't copy-and-paste any code from the previous session, I just looked at it as a reminder of what I needed to do to get the task done. 

I saw this as a 'revelation' of sorts, but something I had always felt was true. The true power of code reuse isn't in the ability to reuse the code. Rather, the ability to reuse steps, concepts and patterns is where the real power resides. 

I decided to write down notes for each kata session. I wrote down the commands I used to progress through the application (Angular CLI commands and .NET Core CLI commands) as well as the names of some third party libraries I used. I felt that these pieces of information would be helpful on any new application.

I decided this was a better approach than trying to memorize each step so I could build out the applications without any assistance. This was a change I wasn't expecting. I thought I would memorize the commands and become so used to them that I could start a new application blindfolded. But I don't need to memorize all the things. I need to work with the tools and practice things to figure out what is important to memorize. 

Some things might change from application to application, depending on the needs of the application. Other things might be dependent upon the version of the tool/technology. For example, building an application with Angular 8 might be different than Angular 10. Of the techniques for using .NET Core 3.1 might be different with .NET 5. So I need to work on memorizing the things that have long-lasting value. It might be best for me to write down the details that change from version to version and focus on patterns and concepts to memorize instead.

### The 'Power' of Design/Requirements

In the first kata session I had to decide what entities and properties to include in my domain model. I had to choose names for my components and folders. I had to decide how I wanted data to be passed between the client application and the backend service. All this took time. And this didn't really help me practice Angular and ASP.NET Core. 

Having well thought out and understood designs and concepts is rarely something developers have when they start a new project. These aspects of a project come with time and continual feedback and learning. There is real productivity gains when such things are clear. It makes me realize how one might be more productive, especially at the beginning of a development effort. Focus on learning and communication and get the concepts and requirements better understood by the team. The code will follow. 

### Learning Lessons

It was interesting to see the different approaches I took. The first session I made some decisions on how I wanted to design to be implemented and then changed my mind in the second session based on that experience. This is what I wanted to learn and practice. I struggled the first time with how to pass data from component to component. I took a different approach the second time through and it was much simpler and easier to implement.

It took me about the same amount of time to get the second kata session finished, about 3-4 hours. But I accomplished much more in that time. The application had more polish to it and I was able to add a few more features. For example, I was able to load the API Path for the backend service from the `environment` object rather than hard-coding it in the service component. I was able to write more unit tests the second time around. 

And again, this is exactly what I had hoped would happen. I learned enough in one session to do things more efficiently the next time. I could get more done in a given amount of time. However, I think for the third time through I will still balance the amount of functionality and the amount of time I take to get it done. Over time, I might want to focus on increasing how much I can get done in a set amount of time. But I also want to be mindful of learning how things work and study different approaches to designing the code, without worrying about how much time it takes.

### Summary

So far this Software Kata experiment has been going as I had hoped. I am encouraged to see how much I can learn. And perhaps switch out the technologies as my skills progress. 

_Never stop learning_

[kata]: https://en.wikipedia.org/wiki/Kata_(programming)
