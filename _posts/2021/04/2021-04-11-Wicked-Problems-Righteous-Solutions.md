---
layout: post
title: 'Wicked Problems, Righteous Solutions'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I had forgotten how _Wicked Problems, Righteous Solutions_ was way ahead of its time. Let me explain.

<!--more-->

### Thanks, Steve

I bought a copy of Steve McConnell's _Code Complete_ in 1996 and my software development career was never the same. It made me aware of the wealth of material out there that I could learn from. Most chapters had recommended readings and I spent the next few years trying to get a copy of dozens of these books. One such book was _Wicked Problems, Righteous Solutions_.

Released in 1990 and written by Pere DeGrace and Leslie Hulet Stahl, the book is rather unassuming. The publisher didn't give the book much help with its soft-cover presentation. It is white with black text and very cheaply made. But as the saying goes, don't judge a book by its cover.

### What is a Wicked Problem?

I have seen a _wicked problem_ defined a few different ways. They all share a core understanding of the difficulty involved:

> In planning and policy, a **wicked problem** is a problem that is difficult or impossible to solve because of incomplete, contradictory, and changing requirements that are often difficult to recognize.

> "a problem whose social complexity means that it has no determinable stopping point"

Source: [https://en.wikipedia.org/wiki/Wicked_problem](https://en.wikipedia.org/wiki/Wicked_problem)

> Horst Rittel and Melvin Webber defined a “wicked” problem as one that could be clearly defined only by solving it, or by solving part of it.

Source: _Code Complete_, 1993 Steve McConnell

In _Wicked Problems, Righteous Solutions_, wicked problems are presented as a critique of the Waterfall method of software development. It states that for problems where the _computer_ is at the heart of the software project, the Waterfall methodology may work. But for problems where the _human_ is at the heart of the project this is rarely, if ever, the case:

> But, we are now encountering problems of a different nature where the computer is no longer at the center of things -the human is- and the machine is now acting to provide of organize information the humans need to produce results.

DeGrace and Stahl define these as "wicked" problems because they cannot be fully understood until after they are solved:

> "The problem is fully understood only **after** it is solved. This means that intermediate results must be obtained **before** a final solution can be reached"

In my day-to-day work I see these types of problems a lot. We talk to a project stakeholder about a new software feature, but they aren't sure about what they want. Or we are asked to find a solution to a problem that no one (in our context) has tried before. Almost anytime we are asked to do something new, we are encountering a "wicked" problem.

As we work in an environment building and supporting and maintaining a software product, we find fewer and fewer wicked problems. But at the beginning, they are everywhere.

### Why They Exist in Software Development

Wicked problems exist in software development because you don't really know if your design will work until you better understand the implementation. So we must find ways to prototype and build solutions incrementally. We _learn_ as we go, gaining knowledge of the solution space (all possible solutions) and focus on where the implementation details lead us. 

As DeGrace and Stahl explain, this is not how the Waterfall method works. It is based on the assumption that you can completely define the problem (through requirements and specifications) before you start with the implementation. We often want to explore a range of solutions and knowing more about these possibilities has a direct influence over the problem itself. It is this interdependency that the Waterfall method struggles with and what makes it a poor choice for so many software development efforts.

### Agile, and Didn't Know It

Within the pages of _Wicked Problems, Righteous Solutions_, you won't find the term "agile" mentioned. Released in 1990, this was almost a full decade before the rise of the Agile Alliance, and Extreme Programming. But the ideas it expresses are signs of things to come and the fundamental shift they bring in developing software applications.

Chapter Seven is interesting in its depiction of "Scrum" and what it might look like _if_ it were applied to software development. It cites the 1986 paper titled "The New New Product Development Game" by Hirotaka Takeuchi and Ikujiro Nonaka. This paper is agreed to be the first instance of "Scrum" being used to define a software development process.

Reading this chapter now, 31 years later, you would think you were reading the intro to a book on using agile methods. They discuss the creation of multi-disciplinary teams and a hands off approach to letting the team self organize. But at the time, a lot of what they described was not common place or well understood. There were very few examples of successful software development projects that used these approaches.

But this is agile software development, even if they didn't realize it. But I suspect they did. They must have known that this 'new' approach was something important and that influenced them to write this book and share their ideas with others.

From my recollection, when I first read this chapter 23 years ago, I was only starting to appreciate the idea that the methods I was taught in school and used every day were not an ideal fit for the problems we were asked to solve. This was 1998 and the software development world was very different than it is today.

I didn't understand what the authors were advocating. They described having a multi-disciplinary team, but I had never seen one before. They described it in hypothetical terms and it just didn't resonate with me.  

Over the next 3 years, the Agile Manifesto was released, I read everything I could on Extreme Programming, unit testing, refactoring and a 'better' way of approaching software development. I never made the connection back to this book and its chapter on "Scrum". Which is too bad, because it was all there for me to find.

While I advocated for the next decade for the adoption of agile methods, it wasn't until 2014 that I worked on my first "agile" project that followed the Scrum methodology. And my software development career was never the same. 

But that story will have to wait for another day.

### Summary

This book destroys the illusion that the Waterfall method of software development could ever be an appropriate response to managing and executing a 'modern' software development project of any size.

It is rarely necessary today to convince someone to use an agile approach to managing a software development effort. But if you ever need some convincing arguments and explanations, this book is a good place to start.

You can see the clear understanding of the "Scrum" methodology and the shape of things to come. I only wished I had paid more attention all those years ago.
