---
layout: post
title: 'Costs of Unit Testing'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
Some people question whether unit testing is worth the cost. Others use the cost of unit testing as an excuse to not write the tests. 

<!--more-->

### A Common Excuse

This week I was speaking to another developer whose current context was very different from mine. I write unit tests for most code changes. It is an integral part of my team's process. We all expect each other to write tests and we strive to maintain a high level of coverage over all the domain logic in the system.

This other developer's context was very different. He was on a small team of two people. Each person relied on the other to review the code changes. I asked him if they wrote unit tests. His response was "We don't have the time or resources to write tests." 

I have used a variation of this argument early on in my career and I suspect a lot of developers have as well. But after 15 years of unit tests I feel like there are very few situations where they are not worth writing.

### Benefits

Thinking that unit tests aren't worth the cost, might be correct... depending on the context. The first thing to understand is the main benefit of unit tests:

> The goal is to enable **_sustainable_** growth of the software project

![Test Goals](/assets/img/test-goals.png)

On the graph, there is a point in the development of a piece of software where using unit tests will allow you to sustain growth. If your project is completed before that point, then writing unit tests might not be worth the effort.

The issue that I see is developers think that point is far off and it is really a lot closer than they think. In my experience, the point when unit tests provide the benefit of sustainable growth usually occurs before a project is completed. 

So I make the assumption that the cost of unit tests will pay off. It means I plan to write the tests and use them to help confirm the code behaves correctly.

### Efficiency

The truth is that writing unit tests does have a cost. The trick is to become efficient at writing tests to minimize this drawback of unit testing. Efficiency comes with practice and proper tooling. And the choice of languages and frameworks have a significant impact as well. For example, .NET Core is much easier to write unit tests for, compared to .NET Framework. 

I have become much more efficient with writing unit tests to the point where it is completely natural to incorporate them into my normal development process. I don't feel that writing unit tests is an additional part of my work. But that is because I have used  this set of technologies and tools for two years. It wasn't this natural or easy at the beginning. But it was the right approach. 

### The Result

Our current situation we have sufficient unit tests to cover at least 70% of our code, roughly 2400 unit tests. It gives us the confidence to add new features quickly. We spend less time fixing bugs and more time adding new features. When the stakeholders ask for changes to the code, we typically don't hesitate because we know the tests will help protect us from breaking an existing feature.

Unit testing might be a cost some teams are not willing to pay, but I have found more often than not the cost is worth it.
