---
layout: post
title: 'Should We Stop Using Pull Requests?'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

My team uses pull requests to communicate changes to each other and help document why we changed the code. Should we stop doing this in favor of something better?

<!--more-->

The use of pull requests was the topic of [a video this week from Dave Farley](https://www.youtube.com/watch?v=ASOSEiJCyEM). He suggested that better techniques like pair programming should be used instead of pull requests. It is a polarizing suggestion. Many people feel strongly in favor of using pull requests. So the suggestion was bound to stir up a bit on controversy. But as with most things, context and point-of-view weigh heavily in how you feel about such matters.

### What are Pull Requests?

A pull request is a technique that software developers use to inform one or more team members that you have a set of changes your would like to be pulled into the code. It was popularized by GitHub users, primarily in support of open source projects. You had a lot of contributors, all from different parts of the world, in different time zones. And not all of them could be trusted to make changes without some type of review by the maintainers of the open source project. So a pull request was a way that the maintainers could review the changes before they were merged into the code. The maintainers could ensure that the changes were appropriate and followed their vision of the project. Some changes would need to be made to the code in the pull request and GitHub furnished tools to make this feedback cycle work. 

### What's Wrong with Pull Requests?

Well, nothing. That is, in and of themselves, pull requests aren't inherently something that you should use or avoid. They are a tool that should be used when appropriate. Each team should decide if pull requests are something they want to use. The issue at hand is that a lot of teams use pull requests and they may find that using pair programming produces better results.

Dave Farley's argument for using something better than pull requests to approve changes to a codebase is sound. The evidence gathered by DORA indicates that in general, teams that commit changes to the codebase at least once a day are more productive. To enable teams to commit changes at that frequency requires fewer barriers to getting changes  merged into the main branch of the code.

Dave suggests that the ceremony of pull requests is, in general, slow. You work on changes, are ready to merge them into the main branch, and then you submitted them to your team mates for feedback. You may have to wait some time to get that feedback. Even on good teams, you may have to wait awhile to get the feedback and be permitted to merge your changes.

Pull requests were born out of the needs of open source projects with distributed contributors that you didn't always trust to do the right thing. People would be working part time on these projects and wouldn't be able to collaborate in real-time with each other. In such a context, pull requests were a great technique for controlling the changes being made to the code in a stable and sustainable manner. But does it apply to your team?

Most teams using pull requests do not work like I just described. They are co-located (even if only via Zoom and Teams) and all work full time at roughly the same office hours. This means that part of what made pull requests a good fit for open source projects is not the same. Are pull requests still a useful technique in other contexts, besides the open source projects I described above?

### What is Pair Programming?

Dave suggests that an alternative to pull requests is the technique of pair programming. It is one of the core techniques of the Extreme Programming approach to software development. It prescribes that two developers pair up to work on changes being made to the code. When practiced properly, the two developers collaborate on the changes, and agree that the changes are appropriate to be merged into the main code branch. 

Many arguments are made against using pair programming. It is not unusual for these to be made by people who have not practiced it. It is natural to not want to change. But I don't think that is what Dave is really trying to convince us of. I think he is arguing that pair programming gives you a faster way of getting feedback than using pull requests. And on that point, I don't disagree. Working with someone in real-time, collaboratively, asking questions, seeing how another person may tackle a problem, is one of the fastest ways to get feedback and to learn. Humans use this technique in almost all aspects of their lives to learn and accomplish great things. So suggesting this as an improvement to the pull request approach is only natural.

But using pair programming can have its drawbacks. When two people pair up to work on a problem, one of the people may be much more dominant than the other. One might be more introverted and not feel like their voice is worth being heard. One person might loose focus and let the other person make a lot of the decisions. In other words, getting two people to truly collaborate could be difficult to achieve in practice. 

### The Role of Context

In my opinion, context plays an important role in how effective any technique is for a team. I've worked on teams where pull requests were mis-used. Only one person on the team could approve requests. And that review was primarily focused on formatting and naming conventions being used, not that the code did something useful and appropriate. I've also worked collaboratively with co-workers very well. I've often said that I always achieve better results when I work with others to get feedback during the development process. Working by myself without feedback rarely produces superior results.

I think that a lot of push back people have for using an alternative to pull requests is simply because they haven't tried it before. There are many techniques, processes, and tools I used to feel were the 'best' way to develop code. But I no longer use them because I experimented and discovered better ways. I'd be crazy to not consider an alternative to pull requests. There may be many ways to get faster (more timely) feedback, and pair programming is one of them. 

### Take a Chance

I like Dave's videos and books. They make me think. They challenge me to be a better software developer. Even when I don't agree, I find that they push me to break out of my comfort zone and take a chance. 

Should my team rely on pair programming more and pull requests less? 

It couldn't hurt to try it and observe the results. 

