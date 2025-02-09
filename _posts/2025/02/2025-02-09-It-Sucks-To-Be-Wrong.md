---
layout: post
title: 'It Sucks to be Wrong'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I often advocate (some would say preach) that progress should be made in small batches. Let's
find some data to back up that claim.

<!--more-->

### Halcyon Days

Starting in the fall of 2018, I had the fortune to work for a special principal engineer. He was in
charge of setting up a series of microservices to enhance an existing off the shelf application. I
learned so much from my time working with him. One practice he instilled in me was to work in small
batches. Pull requests didn't have to be super complicated, large and difficult to review. Make a set
of cohesive changes, back them up with unit tests, and create the pull request. At first it was a challenge
for me to do well. But over time I learned how to work in a manner that made that process feel like
second nature.

Over the next 4 years, I had one of the most rewarding and challenging times in my career. I learned
so many things it is hard to list them all. Messaging, queuing, communication patterns, proper integration
testing, fully automated CI/CD pipelines, and on and on. I switched from using Visual Studio to using
JetBrains Rider and found it to be more productive for me personally. I learned about MassTransit,
RabbitMQ, ElasticSearch, Kafka, OAuth, ODIC, and all the new .NET Core features and capabilities.

In the summer of 2022 I joined a new team that used a completely different set of technologies. All
based in Linux, I abandoned the .NET work and dove into Ubuntu, Visual Studio Code, PHP, TypeScript,
and Postgres. It was pretty humbling to have to learn so much new things. But I wouldn't have missed
it for the world. I came out of it a much better developer. And I had the confidence that I could
devour any new technology that was thrown my way.

This team was pretty special in a lot of ways. They had fully automated pipelines, end-to-end
tests and security checks. They treated code like it was special, and needed to be kept as clean and
tidy as possible. They never settled for second best. If there was a way to make the code better,
they took the time to get things right. When I left this team after 18 months I felt like I had
completed a master class in how to professionally support deployed software. These were the first
people I ever thought of as 'software engineers'.

In this new team I found another special developer. He looked at the code and saw the same things that
Neo saw in _The Matrix_. He was so far better than the rest of us it was impossible not to want to be
around him and learn as much as possible. He taught me how to use `git`. I mean, really use it. Not
the way I had been using it, but to really see the potential it had to be a repository of knowledge,
a way to tell the story of how your software evolved.

This was where I learned how to make a truly clean `git` history. I learned how to use `git rebase`
to make the history of commits make sense. Everyone strived to keep the commits as cohesive as possible.
No one would intentionally mix bug fixes and refactoring changes in the same commit. Each commit had
a simple purpose and the commit message described that purpose as clearly as possible.

### New Challenges

At the start of 2024, I began a new journey with a new team. I was excited to see how all the new
things I had learned could apply to a new context, with new people and new challenges.

We started out as a 2 man team but we grew to 3 then 4 pretty quickly. Now, after one year, we are a
team of 6. I have tried to do my best to have the team consider the practices that served the previous
2 teams so well. Small batches, automated pipelines, end-to-end tests, clean commit histories. My
efforts have had mixed results. We have set up automated pipelines and end-to-end tests and they both
have been very successful. We are an effective team because we have these practices in place. But
the idea of working in small batches and aspiring to have a clean commit history have been a struggle.

### Show Me the Evidence

After failing again to convince one of the developers that working in small batches was going to help
boost productivity, I felt deflated. I couldn't find the words to convince someone to try a different
approach. I thought about it quite a while and eventually struck upon the idea that what I lacked
was evidence. I mean, why would anyone follow my advice if I didn't have any proof to back up my
claims. I could go and cite books and articles to back up my points, but I didn't think that would
resonate as much as actual data from our own experiences.

### The Experiment

There was one repository that had been developed primarily by me. It was back when we were a 2 man
team and I was responsible for 90% of the commits in the repository. What if I compared the pull
requests and git commits from that repository, to say one of the ones where I felt the pull requests
were too large and not resulting in a clean git commit history. I bet I could find evidence to back
up my claims.

I wrote a small program to extract the commits from the repositories. It would calculate a bunch of
data points which I could then use to compare the 2 approaches:

- number of files changed in a commit
- number of added lines in a commit
- number of deleted lines in a commit
- time to review pull request (first approved at - created at)

### The Results

After carefully reviewing the data, it was clear; the data showed no statistical difference
between the 2 repositories. It sucks to be wrong. But the evidence just isn't there.

- The 2 repositories I compared were roughly the same size
- The code in each was developing the same type of application using the same technologies
- They had about 100 commits each
- The main development time was about the same (6 months)

One repository was developed primarily by me, the other was primarily not. However, the data I gathered
didn't show any significant differences:

- The size of each commit was about the same (on average)
- The amount of time to get an approval was roughly the same

### The Conclusions

As a scientist and an engineer, I accept the results and must conclude that my hypothesis
(my way is better) quite possibly isn't correct. I thought my approach would yield (on average) smaller
commits. I thought my approach would reduce the burden on reviewers. But the data didn't support
those claims.

What other metrics might I look at? What about the number of comments on a pull request? I checked and yes,
the number of comments on my pull requests were (significantly) smaller than other developers. But I
can't base any conclusions on that alone. There are so many other variables that could cause that
difference to occur. And besides, the smaller number of comments wasn't because my commits were smaller,
which is what I would have expected.

It is possible that my data is incomplete. Perhaps there are other factors at play here that I could
not control. I wonder if I designed an actual experiment, if some difference would emerge. The DORA
metrics point to stability and throughput as what is important to measure. Maybe that is where I should
look for my evidence.

### Next Steps

While this isn't the end of my investigation, it is a good demonstration of how we can be unable to
base our opinions on actual evidence. It is pretty hard to gather unbiased evidence of a process, a
procedure, a technology or a tool being superior. Keeping an open mind to new ideas is an important
part of being a scientist. But so is questioning any claim someone might make. Changes in context
can result in drastically different outcomes. Be careful what you proclaim to be true. It may not be.
