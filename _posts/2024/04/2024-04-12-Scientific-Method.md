---
layout: post
title: 'Developing Software Using the Scientific Method'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
published: false
---
 
I've wanted to write about this topic for awhile now. I wanted to describe how I use the scientific method to develop software. Turns out it was more complex than I thought.
<!--more-->

### Background

As a software developer, I use a variety of methods to design, code, and test the software that I write. After reading Dave farley's book _Modern Software Engineering_, I became interested in exploring different ways of using scientific approaches to my day-to-day programming tasks. Specifically I looked at how I could use the scientific method to test, debug and analyze the code. 

Part of Dave's message in his book was in order to be considered an engineering discipline we would have to apply more scientific approaches to software development. The one that resonated with me the most was the use of experimentation in the development of software. Exploring the use of experimentation got me more interested in how the scientific method is a useful tool when developing software. 

If you look up various definitions of the scientific method, they all agree on a few key points: The scientific method is a way of acquiring knowledge by testing a hypothesis using experiments and drawing conclusions based on the evidence.

### Developing Software using the Scientific Method

Using the scientific method has many benefits when applied to software development. Because it's only through empirical evidence that you will truly know if your software is working correctly or not. Take for example a small piece (unit) of code. In order to know if it's working correctly or not we write unit tests. Each of these unit tests is a small experiment testing our hypothesis that the code behaves as we expect. Without these tests, all we can do is rationalize about our solution. We look at it and derive that it's correct based on logical reasoning. The trouble with that approach is that our logical reasoning can be flawed in many ways due to the bias we have as humans. 

Another way of using the scientific method to develop software is investigating problems in say, the production environment. A user has reported a problem and you are being asked to investigate it. Based on the report you may think that you know why it's happening. That is, you rationalize your understanding of the behavior. But without performing an experiment to test that understanding, you will never know if you are correct. When you look at software development from a scientific perspective it is important to explain the behavior, but equally important to backup those claims based on empirical evidence.

### Why Doesn't Everyone Work This Way?

Using the scientific method can be very challenging. For one, developing software is an iterative process and you can bounce around quite quickly from hypothesis to hypothesis and experiment to experiment. You acquire knowledge very quickly and incorporate into the next experiment. But rather than taking minutes or hours this can take seconds and as a result you can lose track of where are. We don't typically have an easy way to record our steps so that they can be repeated later. And this is one of the hallmarks of proper use of the scientific method; being able to reproduce the results.

Another aspect of this method that's difficult with software development is the lack of evidence. Sometimes we want to know something about the system that's already happened. But we may not have had the foresight to record those events and the evidence doesn't exist. This makes it very difficult to provide empirical evidence of hypothesis being correct.

And lastly, the scientific method at best is a way of predicting something to be true. It is not guaranteed that it will be true in the future, only that it is likely to be true based on the evidence. This can end up with many false positives and true negatives that complicate our ability to draw conclusions.

But even with all these complications all hope is not lost. You should rely on evidence based inquiry because it is still the best game in town. 

### Evidence-based Development

Here are some of my observations of how best to use evidence based software development.

1. Write unit tests to predict the behavior of your code. Unit tests provide empirical evidence that your code behaves as you predict it to. Each unit test is a small experiment And a passing unit test provides the empirical evidence of our predictions (it works as expected) being true. Trying to make the tests as unbiased as possible and to be very specific about what behavior they are predicting. You have to design your code in such a way that it is easy to formulate hypotheses and predictions about the behavior.

1. Include sufficient logging/tracing in your code so that you can find evidence when necessary. This doesn't mean that you have to know in advance how to capture all of the evidence for all possible hypotheses. But you should use a logging strategy that allows you to add log entries when necessary so that you may gather evidence.

1. Record all of your steps including code changes you tried, queries you ran against the database, the logs that you looked at, and anything else that you may need to use to defend your conclusions.

1. Make every function be unambiguous. In other words, make each function return unique results for each behavior. If two or more behaviors can have the same result then it is very difficult to know which one of those behaviors is truly happening.

1. Use rationalization to determine why you think a particular behavior is being observed. Then use an empirical experiment to gather the evidence to prove that that is correct. If you only act on the first part then you really aren't approaching software development as a scientific and engineering endeavor.

1. Record enough information so that anybody else can reproduce your results. If this is difficult or impossible then you need to redesign that part of the system under observation to make it possible.
