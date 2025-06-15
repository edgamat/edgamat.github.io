---
layout: post
title: 'Best Practices Framework'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

What's the biggest scam in tech that is deemed acceptable? Best practices. Let's unpack that a bit.

<!--more-->

### Context Is King

On Twitter, David Fowler responded to the question "What's the biggest scam in tech that is deemed
acceptable?" with the simple response "Best practices".

Derek Comartin dove into this topic more with a blob post and YouTube video:

What's the biggest scam in software development?  
<https://codeopinion.com/biggest-scam-in-software-dev-best-practices/>

From my perspective, Derek's take was correct. Your context matters. You shouldn't follow a best
practice 'blindly' or assume that you should be following a particular practice because everyone
else considers it so.

Best practices for some might be a bad idea for others, given their context. The constraints for
one group of developers is hardly ever the same for another group. As Derek states, Context is King.
You are constantly making tradeoffs when designing, building and deploying your code. If these
tradeoffs move you away from a "best practice", that is not the end of the world.

However, that doesn't mean that you should ignore best practices. A practice that the industry widely
considers a best practice shouldn't be dismissed as something that doesn't apply to your context. We
should be constantly seeking out new ways to improve our development practices. Learning about best
practices is necessary in order to grow.

As Derek states in his video, the ends of the spectrum are where the problems lie. Practices rarely
should be thought of as something that should never/always apply. Your context will usually place
practices in the middle somewhere. You might find a practice useful and applicable. But perhaps not
100% the same as suggested/described. You taylor things to suit your context. And that's okay.

If we agree that we shouldn't blindly follow (or reject) best practices, how should we determine
which ones to try?

### A Story of Discovery

Not so long ago I worked for a large organization with over 2000 developers. One team I worked on
was tasked with evaluating the current state of software configuration management practices within
the organization. Version control systems had been making their first appearances and we wanted to
know how many teams had adopted them and were they finding their use beneficial. The teams of developers
were not all using the same technology stacks. There were different levels of requirements,
types of applications, timelines, experiences and training across the organization. We quickly discovered
that the current state of CSM practices differed greatly from team to team.

We saw some teams excelling at making changes to their code and deploying it without breaking things.
We wanted other teams to know of these practices and make it possible for them to benefit from what
other teams had learned.

Rather than promote these as "best practices" we chose a different path. We categorized these as
"good practices" and provided each team with a framework for deciding which of these were "best
practices" in their context.

### The Best Practices Framework

The best practices framework provided a means to describe candidate practices and to support people
while they learned more about them. The goal was to guide developers through the candidate practices
and teach them the underlying principles, give them guidance, references, and support for each
practice. We wanted to make people aware of the practices and help them integrate them into their
own process of developing software.

### Evaluation is Difficult

The trouble we have as an industry is a lack of empirical data to back up any such claims about
"best practices". Show me a study that demonstrates (with data) that a team is more productive
when using one practice over another. Such studies rarely exist. Is using one IDE over another
going to improve your productivity? How do you measure productivity? Does using one architectural
pattern over another make an application easier to maintain? How do you define "easier to maintain"?
This inability to point to real data makes most claims of "best practices" difficult to back up,
even when they are "widely accepted" in the industry.

Factors that make evaluation difficult include:

- Many practices are ones where the benefits aren't gained until much later in a software product's
lifetime
- Lack of clear metrics on how to measure improved productivity
- There are other confounding factors that impact a practice's effectiveness

### Start With Good Practices

In our best practices framework, we curated a list of candidate practices. At first, we included
several practices that we all agreed were "widely accepted" in the industry. But we soon discovered
that we had little practical experience with these practices. It was difficult to provide support
and guidance for these.

We shifted our focus and looked to the teams that had the best outcomes. Teams that deployed code changes without
breaking things and could easily determine what code was deployed and who changed it last. We documented
these practices as our first set of candidate practices.

We labelled these candidate practices as "good practices", as this seemed to be the place where any
best practices would come from. Not all teams were using all these practices. Some didn't have universal
appeal to all teams. But having the list of all practices gave teams a chance to see what was available
and look through samples and supporting documentation. Some of them would then decide to try some of
them and evaluate their effectiveness.

Some of these good practices would not make sense in other organizations. Our list included ways to
manage the changes being made to mission critical systems using in-house tooling. It included practices
for communicating changes via the tools we knew everyone had access to. These practices only made
sense within the organization. It is doubtful we would ever claim them to be "widely accepted" outside
this context. But for our organization, they made a lot of sense.

### How to Evaluate A Good Practice

Evaluating any practice is challenging. Even ones that everyone agrees to be worthwhile. A scientific
approach would require experiments and data gathering. Empirical evidence is a must. But none of that
exists or is too costly to generate on our own.

The annual State of DevOps Report is an attempt to provide some scientific basis for the practices
we follow. I won't go into the details, but the evidence they gather is mostly through surveys. The surveys
ask respondents questions about metrics that may be good predictors of effective teams.

But these questions focus on DevOps practices. Lower level design and architecture choices are not
considered (yet). Nor are programming languages, coding conventions, tools or processes measured. Do
we have any data to suggest that choosing any of these is better than any other? How would we design
an experiment to evaluate one architecture approach versus another?

Unfortunately, there are no easy answers. But perhaps an empirical outcome is not necessary. Here
are some other ways we can gauge the effectiveness of a practice:

- Compare outcomes between teams (One team adopts a practice, the other does not). While not scientific,
it can be useful none the less.
- Try defining a goal to achieve before adopting a new practice. Like reducing the number of bugs,
or improving the team's output. See if adopting the practice influences those metrics.
- Use surveys and retrospectives to gather feedback from the team. How has adopting a practice changed
things? Were adjustments made to make it more effective in our context?

It is important to re-evaluate the effectiveness of practices. Things change over time and the context
of a given choice could men something isn't as effective as it was. Or something that was decidedly
unattractive is not worth while considering.

But remember, Context is King. Look at each practice and decide if the ideas it affects are meaningful
in your context. Do we have the problem this practice addresses? If not, then it is fine to focus on
other things.

### Summary

Adopting widely accepted practices may not have the desired impacts if you don't provide the necessary
support. Establishing a framework for selecting candidate practices is a start. Be careful when evaluating
the effectiveness of a practice as this can be very difficult.

Question everything. As engineers, we should be open to new ideas, and should not be dogmatic about
anything.

Context is King.
