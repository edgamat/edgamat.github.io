---
layout: post
title: 'Software Kata Part 1 - Angular'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

The concept of [kata][kata] in software development has been around for years. The idea is to memorize and improve your coding skills through repetition and practice.

<!--more-->

### Background

I find that some technologies I use every day while others I use less often. I spend time re-learning a portion of them each time I start a task using the technology and I find that both frustrating and wasteful. 

For example, I may not start a new software application from scratch very often. So when the time comes I have to re-learn some of the steps to get a new project up and running. Or another example is when I learn a new technology (like Angular) and don't use it steadily. It may be weeks or months between tasks that use it. In either case, I feel I need to find ways of practicing these skills more frequently.

### The Process

I'm going to start with a simple plan. At regular intervals (yet to be determined) I will practice a set of skills for a given technology. I'll have a pre-defined script of steps I want to complete for each. The script will start small and over time will hopefully grow to include more steps as I become more used to the technology. 

I plan on creating a folder structure on my computer where I can practice. For example, here is where my Kata for Nov 24, 2020 would be for Angular:

```powershell
C:\code\kata\2020\11\24\ng
```

I've decided to begin this new process with Angular. It is a technology that I have recently learned for my job and I don't feel like I get enough practice. I've decided to start with performing this kata once a week. I may find that is not frequent enough. Or it might be too frequent. I'll adjust the schedule if necessary.

### The Skills to Master

So what is the initial script I want to practice? The basics of course, but specifically:

- I want to practice creating a new project and build out the start of a simple application. 
- I am a fan of the [Bootstrap][bootstrap] UI framework. I'd want to incorporate this framework into my script. 
- I'd also want to incorporate support for simple HTTP calls with loading semantics. In other words, when an HTTP call is made, have a loading spinner overlay the current UI so the use knows the application is loading the requested data.

I think that would be enough. What should the concept be? Several single-page application frameworks have templates available for things like 'to do' applications or 'weather forecasts'. I want something similar to this.

Let's start with a simple dashboard. It will display three charts of data: temperature, pressure and oxygen levels.

- Pressure 12.48 (PSI)
- Oxygen 20.79 (%)
- Temperature 21.15 (C)

The data will load from an HTTP API. The dashboard will include a refresh button to reload new data.

### Summary

I hope this experiment will be successful. I want to retain the new-found skills I have with Angular and to grow them stronger with practice. The repetition of the kata will be important in the that goal. I hope to report my progress with a future blob post to see how it is going.  

[kata]: https://en.wikipedia.org/wiki/Kata_(programming)
[bootstrap]: https://getbootstrap.com
