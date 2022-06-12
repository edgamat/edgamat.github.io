---
layout: post
title: 'A Gentle Introduction to Pair Programming'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I recently switch to a new development team at the client where I am working. The new team uses pair programming to develop changes to the software. My introduction to this technique has been a good one. 

<!--more-->

## What is Pair Programming?

Pair programming is a technique where changes are made by a pair of programmers working at the same computer. One person drives (ie. has their hands on the mouse and keyboard and makes the changes) while the other observes and guides the changes. The roles each person plays should switch frequently. The premise of this approach is to increase software quality. 

It is debatable if the quality of the software improves using pair programming. Some teams are more effective at it than others. Some individuals are better at it as well. The context for your project/team can also greatly impact how effective it can be. But for some teams it is a good technique to use when developing software.

## A Gentle Introduction

When I joined the new team, I was paired with a teammate to learn more about the system and to get my local workstation setup for developing changes to the software the team supports. Was this pair programming? In a way, yes. But not really. We weren't making changes to the code. 

But soon enough I was able to watch a pair of teammates work on a change to the software. One person was driving and the other was engaged in asking questions to the driver, looking up answers to questions the river had and reviewing the code changes being made. Sometimes the observer would spot syntax errors earlier than the driver. He would mention them to the driver who would make the necessary corrections. Other times the driver would get stuck and talk through where to go next with the observer.

They broke for lunch and when I re-joined them a couple hours later, the roles had reversed and the driver was now the observer. It was interesting to see how different the two developers worked. They used different editors for making their changes for example. But the flow of pairing together was the same. One supporting the other.

I was impressed with how much it was a very collaborative engagement. In my experiences, engagements of two or more developers would result in a dominant voice leading the way. It was rare to see each person contributing equally to the task at hand.

This was the type of introduction I needed to pair programming. I was able to see how it is done well. I know if I had tried to pair program with someone else who was not familiar with it, we would have not done as well.

## Is Pair Programming Less Productive?

This question has been raised a lot by people I have met who object to pair programming. My own bias towards pair programming has leaned in the "no, it can't be as productive" side of the debate. Two developers working on a single task cannot be as productive as each of them working independently on separate tasks. 

This is similar thinking to unit testing. Writing unit tests is going to slow you down and make you less productive, being the central argument. The same could be said of Test-Driven Development. One could argue that it slows down the development process. But I'm not as convinced of either of these ideas as I used to be.

The goal of unit tests is to enable **_sustainable_** growth of the software project.

The goal of test-driven development is to improve the quality of the design.

These techniques don't have an immediate payoff. Their worth grows over time as you build more and more software. The same can be said for pair programming.

Just by watching two developers effectively pair program I can see some tangible benefits, beyond the code that is written. First, it is a great way to learn. Each person while pairing is learning from the other. Each person has a shared understanding of the new behavior being added. This knowledge is not typical amongst teammates when only one developer is working on a story. Second, it is a great team builder and means of communication. Working with each other builds trust and understanding between teammates and the flow of information is increased as well. These are essential in today's world of remote work and pair programming can overcome some of the downsides of not being able to be in the same room when working on a project. 

## My Journey Begins

I was able to work with another developer on the team for a few hours this week. Our pair programming session was engaging and (from my perspective) productive. I felt at time we were going slower than I would have liked. I think that was due to my inexperience with pair programming and with my unfamiliarity with the work being done. Both of which will improve with time. But my initial observations make me realize there are some valuable lessons to be had from this approach. 

First, patience is the key. If you are the type of person who gets frustrated when you aren't in control, you are going to struggle with pair programming. When done properly, it is a collaborative process of equals. And that can be tough for some folks to adjust to. I'll admit, I am one of those people. But the team has shown me a lot of patience so far, and I owe them equal patience in return.

Second, remote pair programming has its challenges, but it isn't impossible. This new team I have joined proves that it can be done. The remote tools are much more advanced for online collaboration that ever before. You just have to take advantage of them and learn to use them properly.

I am looking forward to this new experience. I hope it makes me a better programmer and I hope I can be a productive member of this team.

-----

As a side note, I was glad to find the original Extreme Programming website is online after all these years. I used it to form some of the ideas I wrote in this post. Check it out at [http://www.extremeprogramming.org/](http://www.extremeprogramming.org/).



