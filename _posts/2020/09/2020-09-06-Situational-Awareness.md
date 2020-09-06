---
layout: post
title: 'Situational Awareness'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I feel that decision making requires a good sense of Situational Awareness. This applies to a lot of situations, including software development. Let's explore it, shall we?

<!--more-->

### What is Situational Awareness

Situational Awareness (SA), according to Wikipedia, is:

> ...the perception of environmental elements and events with respect to time or space, the comprehension of their meaning, and the projection of their future status.

The definition consists of three parts:

1. The perception of your environment
2. Comprehending the situation
3. Projecting the future state of your environment

Most uses of Situational Awareness involve threat assessments. Assessing a situation where there is a threat to life or property, such as a terrorist threat, an attack, or something dangerous like a storm or natural disaster.

How does any of that apply to software development?

### Perception of Your Environment

In most situations, it is important to know what is going on around you. When you are developing software, you need to know the status of the things you are working on. If working with a team, this includes the state of your entire team's tasks and assignments. Can you tell when someone is ahead of schedule or struggling with a task? Do you know if you are on track to meet a commitment? Is someone waiting on you to review their work?

Equally important is to know what work remains to be accomplished. Do you know what your next task will be? Do you have enough information to start the next task when the one you are currently work on is complete?

It is also critical to know the state of your software. Is the production environment online and operating normally? Did your last set of changes get merged into the main code trunk successfully? Are all the automated tests running successfully? 

The perception of your environment is critical in making good decisions. If you don't have a means to see all these aspects of your environment, then that is the first place to start. Identify the areas you need more visibility and work on gaining the information you need.

### Comprehending the Situation

When developing software, one of the tools developers use is pattern recognition. We look at the code and try to see if there are patterns that exist or that we might apply to a situation. The same skill can be applied to situational awareness. Are there patterns in the environment that we have seen before? How should we interpret and comprehend the cues we see in our environment?

This comprehension is made easier having a good perception of your environment. But it also is easier through experience and practice. Let's look at a few situations that may indicate a problem exists in our environment.

- The latest code changes are causing some of the automated tests to fail.
- The next task you are to supposed to work on has some missing detail or unanswered questions.
- One of your co-workers is behind schedule on their current task.

We always need to guard against a false comprehension of our project's environment. What we think might be happening may not accurately reflect reality. How you read a situation is influenced by a great many things. For example, I often feel time pressures that don't exist. And I sometimes fail to recognize changes that I'm about to make are going to cause existing unit tests to fail. 

When we are under (perceived or real) pressure we can often make poor decisions. It is therefore important to verify assumptions and have good communication to avoid any misunderstandings.

### Projecting the Future State

Now that you've spent time comprehending your situation and interpreted the current state of things, you need to think about the future state of your environment.

This is the tricky part, obviously. It can be difficult predicting where things will end up. This too is where experience comes into play. You will have learned over time where things may end up and you can take steps to avoid these situations.

**Example:**  
The latest code changes are causing some of the automated tests to fail. We know that this will prevent us from deploying the current code to the testing environment.

**Example:**  
The next task you are to supposed to work on has some missing detail or unanswered questions. We know this will cause a delay while the tasks is made ready to be worked.

**Example:**  
One of your co-workers is behind schedule on their current task. We know this will require other tasks to be delay and some commitments may need to be adjusted.

### Taking Steps to Avoid Problems

Being aware of your (or your project's) situation when developing software is important because it can give you a chance to change the outcome. If your current task, or project, is going to fail or cause delays, you'll want to do something to avoid that outcome. And the sooner you can see the signs, and know their bad side effects, the sooner you can take preventative measures.

Sometimes that might mean bringing up the concerns at your next daily standup meeting. Other times, it might mean you simply need to talk to your co-workers or your client and get some additional information. Whatever the action you feel is appropriate, being more aware of your situation gives you more time to make things right.
