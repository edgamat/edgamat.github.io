---
layout: post
title: 'ASPNET Core Unit Testing Lessons Learned Part 3'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When building a Web API endpoint with ASPNET Core, you end up writing a lot of tests. Along the way
I found some ideas that helped me manage them more efficiently. Let's take a look at a few. Note, that
most of these apply to any testing effort, not specifically to ASPNET Core projects.

<!--more-->

## System Unit Test


## Refactor, Refactor, Refactor


## Issues and Observations

As a lesson learned, I recommend having helper methods that create the system under test, i.e. the class
whose behavior you are testing. It guards against duplicate setup code and guards against brittle tests
as the constructors of these classes evolve.

When test classes contain multiple test methods, look for opportunities to refactor the code. Just like 
production code, test code can suffer from code smells, duplication, poor naming conventions, and so on.
Keep the test classes neat and tidy and they will continue to provide value as the code evolves.

