---
layout: post
title: 'Reviewing Programming Pearls, 21 Years Later'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Are the principles laid out in the book _Programming Pearls_ still relevant today? Let's find out.

<!--more-->

### Background

Last year, a lot of workers were sent home to work due to the COVID-19 pandemic. In my case, I had to look over my bookshelf and decide which books to bring home with me. That was a difficult decision for me, and I ended up leaving many at work. A few months went by and I found myself wanting to refer to several of these books so I went back to the office and retrieve a large number of them. Unfortunately, I didn't have the space at home to have all these books. So I decided to review them and see if I could donate some and cut down on the ones I keep. 

Fortunately some of them are available in digital eBook form, so I could simply switch to an eBook rather than paper. But some of them aren't cheap and I didn't want to spend money on an eBook unless I really felt it was worth it. 

The first such book was _Programming Pearls_, 2nd Edition by Jon Bentley.

### The Book

Released in 1986, _Programming Pearls_ was one of several books recommended by Steve McConnell in _Code Complete_. Released in 2000, the 2nd edition of _Programming Pearls_ was available to me and for the first time I had the opportunity to read this book by Jon Bentley.

Originally presented as a series of essays in the _Communications of the Association of Computing Machinery (ACM)_, the 2nd edition included 3 new essays and substantial revisions to the 13 essays that appeared in the 1st edition. Each essay focuses on a programming problem and teaches programming techniques and fundamental design principles. 

This gave me a moment to pause:

> This edition reports the run times of many programs on "my computer", a 400 MHz Pentium II with 128 megabytes of RAM running Windows NT 4.0.

I think at the time I also had a Pentium II with 32 megabytes of RAM. Oh how times have changed (my current computer has 32 _gigabytes_ of RAM).

### The Principles

Each essay lists one or more principles and this is where the insights can be found. Even 35 years later, most of these are still applicable today.  The first essay is an excellent example. It is a case study of sorting a file on disk. The first principle for this problem is "careful analysis of a small problem can sometimes yield tremendous practical benefits.". He sites Chuck Yeager as praising an airplane engine: "simple, few parts, easy to maintain, very strong". Bentley sees these as good attributes for a program. I tend to agree.  The second principle is to define the right problem. So often that is the biggest challenge; solving the right thing! And one other that speaks to me is the principle of "A Simple Design": 

> Simple programs are usually more reliable, secure, robust, and efficient than their more complex cousins, and easier to build and maintain.

Designing simple solutions is quite difficult in practice. I have met developers who are much more talented designers than me and they always seem to design much simpler programs. But it is a skill that can be learned with practice and experience. 

And this is only the first essay. There are similar principles throughout the remaining essays. For example, here are two from the 5th essay:

_Coding._ I find it easiest to sketch a hard function in a convenient high-level pseudocode, then translate it into the implementation language.

_Testing._ One can test a component much more easily and thoroughly in scaffolding than in a large system.

In the the essay, Bentley again emphasizes the simple design approach, using a quote from Gordon Bell:

> The cheapest, fastest and most reliable components of a computer system are those that aren't there.

And again in the 7th essay (attributed to Albert Einstein):

> Everything should be made as simple as possible, but no simpler

Reviewing these essays, it is clear the common thread through most of them is the concept of simplicity. Simple designs were discussed at great length when Apple products made "simple" a marketing slogan. Apparently way ahead of his time, Bentley shows us through numerous examples how much this can affect the quality and maintainability of software programs.

### Relevance

I find much of what Bentley discusses to be still relevant today. The examples in the book are somewhat dated I grant you. The modern software developer may not need to sort a file on disk or squeeze performance or memory usage from a program. But the thought process and techniques are universal. They still are very useful and can make you think differently about the tasks you are a working on. 

### Summary

I ended up getting the eBook format of this great book. The paper copy is not leaving my bookshelf yet, but I am glad to have a digital copy. I imagine I will refer to it again in the future.


