---
layout: post
title: 'The Mythical Man-Month - A Review'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
published: false
---
 
Written in 1975, it was 20 years old when I bought my copy of _The Mythical Man-Month_ 25 years ago. What can a book more than 45 years old tell me today? Quite a lot actually.
<!--more-->

_The Mythical Man-Month_ was written in 1975 by Frederick Brooks. It is a series of essays on software project management based on his experiences working on large software development projects. His central argument is that large programming projects suffer management problems different from small ones due to the division of labor. I rarely work on 'large' software projects but I can attest to experiencing some of the same problems more than once in my career. But when I was first introduced to this book I hadn't had very many experiences at all. I was only a couple years into my career and was still learning the basics.

But even small projects can have 'division' of labor issues. The 'Agile' movement, and the popular process methodologies like Scrum, has a lot to prescribe on how teams are formed and work is accomplished. And this makes some of the ideas and arguments in these essays very applicable, even today.

### The Myth

So let's start with the titular "Mythical Man-Month". Brooks observes that the man-month as a unit for measuring the size of a job is "a dangerous and deceptive myth". A man-month is often used on large programming efforts because it help planners get a sense of how many people they will need and how much time it will take to complete a task. The questions usually start off as "How much time will it take a programmer to do X?" Let's say the best estimate at the time is 9 months. The "myth" is thinking that the number of programmers and the number of months are interchangeable. It is rarely possible for 9 programmers to complete the same work in a single month. As Brooks points out "the bearing of a child takes nine months, no matter how many women are assigned." 

I have seen this myth take hold on a lot of larger projects. This sort of thinking gets us into trouble quickly because it is wildly optimistic. It assumes that if you assign more people to a task it will get done more quickly. That in and of itself is not usually wrong. But thinking that adding more resources will significantly reduce the time the work takes is usually where the problem lies. Brooks argues that due to the increased inter communication of the people assigned to a task, that adding more people will often lengthen the time a task takes to complete not shorten it.

It is interesting to see how such ideas have evolved into the way we develop projects some 4 decades later. Some of these issues are ones we still face. Agile methodologies are more commonplace today and the way we form teams to accomplish programming tasks is a lot different as well. But we can still fall victim to the "myth" when trying to find ways of accomplishing tasks more quickly by adding more people.

### The Contents

I have some favorites among the essays. Chapter 4, "Aristocracy, Democracy, and System Design" is a great discussion of "Conceptual Integrity". It is something that even today is a difficult thing to achieve. Chapter 5 defines "The Second-System Effect" which occurs when a programmer attempts to build something for the second time:

> The general tendency is to over-design the second system, using all the ideas and frills that were cautiously sidetracked on the first one.

This 'effect' has always been in my mind as a developer over the years. It doesn't just apply to 'systems' but also just to programs and modules in general. I can't tell you how many times I have created the second design of something and wanted to 'improve' the design from the lessons I had learned building the first one. But I remember the "second-system effect" and try not to embellish things too much.

Chapter 11, "Plan to Throw One Away" is another one I like. It admits that when designing and building software, it is common for the first thing we build to be less than perfect. While building the first thing, we learn so much and that feedback is worked into the codebase so much so that often the original design is changed significantly. It is so frequently done, that the author suggests that you should plan to throw away your first attempt at a solution, you will anyway.

In Chapter 18, the author revisits this idea (20 years after proposing it) and suggests the idea is tainted by following a waterfall methodology. I tend to agree, but that doesn't invalidate the central premise. It is the idea of failing to hit a target with your first attempt and having to redesign it as you learn more information. This is one of the strengths of modern agile methodologies. We'd be wise to plan on redesigning the system as we go, rather than attempting to get the design perfect before we write any code. 

One addition to the 20th Anniversary edition I own is the "No Silver Bullet" essay. I still remember going to the university library to photocopy the essay from _IEEE Computer_. It had a profound influence on the software development industry. Written in 1987 it contains many arguments, ideas and suggestions that hold true today.

### The Legacy

I re-read the book recently to support writing this blog post. It is amazing how much of its contents are meaningful and applicable today. It pre-dates the mainstream adoption of agile methods for developing software. But central in its themes are ideas of growing software, incremental design, and the challenges we still face today dealing with the complexity of software development.

I think any developer I know today could benefit from reading this book. There are few parts that refer to 'ancient' technology and even when they occur, it never distracts from the ideas being presented.

It won't be too long before the 50th anniversary of its publication comes around. I hope our industry takes the time to pause and recognize the great contributions these essays have made to the generations of programmers who have benefited from it.
