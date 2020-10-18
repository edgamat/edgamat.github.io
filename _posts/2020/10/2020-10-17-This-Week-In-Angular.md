---
layout: post
title: 'This Week in Angular'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

This past week I spent a good portion of my time working on an Angular project. It is only the second Angular project I have worked on so my experience is limited. But I wanted to write down some thoughts on what I felt using this technology. My future self might like to remember what it was like.

<!--more-->

### Similarities Abound

Having not worked a lot with single page application (SPA) frameworks before, I expected there to be a lot of challenges using one. But so fare (granted only two projects here to draw conclusions from) I have found more similarities than I expected.

First, we are using HTML, CSS, TypeScript. I see myself faced with the same challenges here in an Angular app than, say an ASP.NET MVC app. It takes work to craft the HTML correctly. It requires time to get the layout and behavior the way you want. One advantage I have found is in Angular the CSS is automatically scoped to the current component you are working on. It seems to cut down on the problems you face where one CSS change for one component affects the layout somewhere else on the page. A nice benefit.

Second, we are using Bootstrap and Font Awesome. It wouldn't matter to me if we were using Material Design Components instead. The takeaway is the ue of these frameworks for building an application are no different in Angular than a MVC website rendering HTML on the server. But of course, that means that the same problems exist too. People (including myself at times) dream up some pretty interesting ways to abuse the CSS frameworks and generate some horrible HTML in the process. I've been fortunate to have worked with some talented designers who have the gift of building user interfaces with a minimum of waste and complexity. It has helped me be a better UI coder. And these skills are equally important in Angular ans with any other HTML web-based UI.

### Powerful Prototypes

I was analyzing a user story in the backlog this week. I decided to build a prototype using Angular to learn more about the scope of the story. I built a new app from scratch using the Angular CLI and created the prototype in no time at all. It was very useful as it showed me how to build out the behavior I was looking for and I learned a lot about some of the details that were missing from the story's description and acceptance criteria.

But I was surprised at how easy it was to create a working UI prototype in Angular. The hot reloading capabilities meant I had much quicker turn-around time when trying changes. And building out components using in-memory API calls to get data meant I could simulate most of the visible behaviors for the application. I was very impressed. 

I ave heard other developers say how building out a prototype using React was easy and quick. But I suspect that most SPA technologies (Angular, Vue, etc.) also have the ability to create functional prototypes much more easily than server-side based ones. I plan on doing more prototyping with Angular moving forward. Not only because it is useful for planning and learning about the tasks I am working on, it also helps me learn more about Angular and using the tools to build web applications.

### Component Communication

I am taking an online course on [Angular Component Communication](https://app.pluralsight.com/library/courses/angular-component-communication) and it couldn't happen at a more opportune time. I learned a lot this week on how to use `@Input()` and `Output()` decorators and I even used s service to pass 'state' from one component to another.

Until this week, there was a lot of mystery about this part of Angular for me. But using the course material as a guide, and actually working on 'real' code using these techniques, a good portion of that mystery has been revealed. I'm not 100% sure I know enough to use these techniques effectively, and I'm sure I'll continue to make mistakes and learn. But I feel like using the built-in techniques for communicating between components is not going to be a problem moving forward. 

### State Management

When someone first introduced the idea of 'state management' in an Angular app, I was confused. I didn't understand the concept, nor did I understand the solution they were describing. It was a 'redux' pattern solution using [NgRx](https://ngrx.io/). It was like all my years of software development were no longer useful. The need for 'reducers', 'actions' and 'selectors' to manage state seemed strange at best and irresponsibly complex at worst.

In a similar way to learning more about component communication, the online courses I've been taking helped me get a better handle of state management. These courses showed solutions for state management that did not use NgRx. These 'simpler' solutions had much more applicability to me for the size of the projects I had been using. I can see where using NgRx would be more applicable on larger projects. But it is a huge undertaking to commit to using it. I would only use it if the other approaches no longer made sense for the project.

### Wrap-Up

I think I learned more about Angular in this past week than I have in the past two years of on-and-off studying and tutorials. I am looking forward to learning more in the weeks to come. Even if the project I am working on aren't focused on Angular any more, I feel like learning Angular will serve me well in the future. I think it will also help me learn more about React and Vue and other single page application frameworks. Time will tell.
