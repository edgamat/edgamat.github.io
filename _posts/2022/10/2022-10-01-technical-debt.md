---
layout: post
title: 'Is Technical Debt Misused/Overused?'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Technical Debt is a real thing. But sometimes it is used to label things that are quite different. 

<!--more-->

### What is Technical Debt

Here are some definitions that I have found:

> "Technical Debt" refers to delayed technical work that is incurred when technical short cuts are taken, usually in pursuit of calendar-driven software schedules.

_Managing Technical Debt_  
_Steve McConnell, CEO and Chief Software Engineer, Construx Software_  
_Version 1, June 2008_  

> Technical debt (also known as tech debt or code debt) describes what results when development teams take actions to expedite the delivery of a piece of functionality or a project which later needs to be refactored. In other words, it’s the result of prioritizing speedy delivery over perfect code.

https://www.productplan.com/glossary/technical-debt/


> When taking short cuts and delivering code that is not quite right for the programming task of the moment, a development team incurs Technical Debt. This debt decreases productivity. This loss of productivity is the interest of the Technical Debt.

https://www.agilealliance.org/introduction-to-the-technical-debt-concept

There is a common theme running through these definitions. They all describe situations where a team delivers a solution that is not up to the task at hand, with the expectation that a proper solution will be done later. Sometimes the team realizes this is happening and agrees to take on this addition future work (i.e. take on the debt). Other times, the team doesn't realize this until later (surprise!).

### Misuse/Overuse

In my experience, development teams sometimes misuse, or overuse, the metaphor "technical debt" to describe many other types of tasks that they feel need to be done.

For example, I have seen teams create lists of "technical debt" that include items such as:

- fixing bugs
- upgrading external dependencies (e.g. upgrade from v1 to v2 of a tool everyone uses)
- changes due to new technical requirements (e.g. move the database to a new server)
- address security vulnerabilities
- anything that isn't a behavior that the users can observe
- routine maintenance of the system

Strictly speaking none of these are technical debt. At least not compared to the definitions listed above. So why do teams refer to such items this way?

### A Possible Process Breakdown?

One reason is that it makes it easier to manage work that the team needs to do, but struggles to define within their normal work stream. On a project that follows an agile process (like Scrum) product backlog items are driven by the product owner. When a 'technical' story needs to be written, it can be difficult to write the story so that it represents 'value' in the same way that 'normal' stories do. The product owner and the developers agree that such tasks are to be categorized as "technical debt". The description and acceptance criteria of such items are different and how a team considers them 'done' is also different.

One could argue that this is a breakdown in the process and this is not the way to handle technical stories. But adhering to some ideal agile process is missing the point. It is work that needs to be done and the team and their product owner have worked out a process that works for them. Who are we to say that their approach is wrong?

One risk is that this approach could lead to abuse. Developers might put items in the "technical debt" list because they don't know how to express the business value the work represents. Or it may be something the team feels needs to be done sooner rather than later and they know that they will have more of a say in getting technical debt items prioritized. 

Another risk with this approach is that technical debt items can sometimes be tracked separately from the normal product backlog. This can lead to important technical items being forgotten about. These technical debt items need to be represented as equals with other backlog items during planning sessions. That means they need to be in the same list as other backlog items, and their relative importance properly discussed with the product owner.

### Let's Talk

I have found that items teams call "technical debt" are usually items that the developers are advocating for. They don't originate from the product owner but rather from observations made by the developers. It is convenient for teams and the product owner to discuss these items differently and using "technical debt" to describe them is something a lot of teams use. Honest and open communication is the most important part, not if they are truly "technical debt" or not. Developers and product owners need to decide on their relative importance and get the work done. Don't get hung up on what they are called.

**NOTE** I have been on teams that have perhaps misused the term "technical debt". I didn't just see a team do this sort of thing, I was an active participant! But sometimes you have to pick your battles. Getting a team and product owner to use the metaphor of technical debt more strictly is not one I feel is worth fighting. 
