---
layout: post
title: 'Why Do We Write Unit Tests?'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

A lot of time we get wrapped up in our day to day work and forget to be mindful of the reasons why we work a certain way or follow a particular approach to software development. One such practice is unit testing our code. It is important to remember why we test our code and how to do it well.

<!--more-->

## The Ideas

One justification for unit testing I have seen used is its ability to verify the correctness of the software. In my experience this rarely ends well. Focusing on demonstrating correctness is near impossible due to the overwhelming number of test cases that would be necessary to achieve such a goal. You would need to test so many combinations of inputs and paths through the code in order to make it effective. And even then, could you say the code is "correct"? 

Another justification (which I have used at times) is that unit testing is there to help discover problems. When done properly, a good suite of unit tests can discover problems with the code earlier in the life of the software, which is almost always easier (and quicker) to correct.

While I find this to be true on most projects, it is short-sighted, or perhaps limited in scope. People should gain the benefit of finding problems earlier when writing unit tests, but there has to be more to it than that.

Advocates of Test-Driven Development (or Test-Driven Design) see the writing of unit tests as a design technique. Before writing the code, writing tests that demonstrates the expected behavior is a powerful means of expressing your understanding of consumer's intent. Showing how someone expects to use the code gives you a much better understanding of how to go about implementing it. 

So where does that leave us?
- Do we write tests to confirm correctness?
- Do we write tests to discover problems?
- Do we write tests to help design the code?

## The Goal

In the book "[Unit Testing: Principles, Practices, and Patterns][ppp]", Vladimir Khorikov describes his motivation:

> The goal is to enable **_sustainable_** growth of the software project.

He explains that over time the codebase will degrade without the support of unit tests. You'll spend more time fixing bugs than providing value to your customers.

I like this explanation. For me it promotes two key ideas for why we _need_ to write unt tests for our code. First, writing unit tests is the best known way to sustain and grow your code over time. I 100% feel this is true and anyone who cares to argue against it is welcome to try. And second, it is pragmatic in that it doesn't advocate any particular approach.

Sometimes, I write tests to confirm the correctness of the code I am writing. I need to be sure the logic is rock solid and has no known flaws. So having '100%' coverage of all code paths in a given portion of my code is critical. 

Other times, I write a series of unit tests to protect myself from introducing a bug in the future. I want the tests to act as a safety net in case I, or someone else, changes the code's behavior.

And I'd be remiss if I didn't admit that writing unit tests help me design better code. When I am not sure of the behavior of the code, I find it incredibly helpful to write test cases as a way of discovering the desired behavior. I dance back and forth between writing tests and evolving the code based on what I discover.

I do however feel that none of these justifications is absolute. I cannot advocate that you should always write tests to confirm correctness. Nor can I say that all tests should be written with the intent to discover problems or to help deign your code. Real world testing and coding is rarely going to produce any approach with such universal applicability.

## Summary

I'd rather like to think of writing unit tests as a partner in my pursuit of providing value to my customers. I use them any way I can to continue to grow, learn and discover what the software needs to do in order to solve the needs of my clients. 
