---
layout: post
title: 'Using Code Inspections the Wrong Way'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
published: false
---

Modern IDE tools provide a wealth of productivity features including automated code inspections. These analysis tools run in the background and give you advice (suggestions) or raise warnings (or errors) when they discover a problem. How you react to them is important. Let's see how not to do it.

<!--more-->

### Resharper, StyleCop and Visual Studio

For a number of years now, Visual Studio has included several code inspections to assist developers writing in C#. A popular extension is to use StyleCop to help highlight coding issues. It provided compiler warnings or errors when it encountered issues. The list of rules it followed were supposedly a set of 'best practices' from Microsoft.

Another popular extension is Resharper from JetBrains. In addition to a number of other features to increase productivity, Resharper includes a number of coding inspections, similar to StyleCop. The Resharper rules are thought to be a set of 'best practices' based on the engineers at JetBrains. Some of them are identical to the ones Microsoft promotes, while others are unique to JetBrains. 

The end result in either case is that you have a number of ways for your IDE to provide you feedback with code inspections. These suggestions, hints, warnings and errors are there to help you identify and correct potential issues with your code.

### The Wrong Way

So here is what not to do. Ignore these inspections. Or worse yet, assume they all are applicable to your code and need to be 'fixed'.

Ignoring these inspection results can be very easy. After all, they are just 'suggestions'. But ignore them at your on peril. These inspections are included by JetBrains and Microsoft for good reasons. They are issues with the code that typically will cause problems. So it is important to recognize they are present in your code base and to have a plan to address them.

At the opposite end of the spectrum, you could review each and every suggestion, hint and warning and 'fix' them all so the code inspections do not find any issues with the code. In my opinion, this is just as wrong of an approach as ignoring them.

### A Better Way

I believe a better approach to dealing with the code inspection results is to understand each one and figure out if its something you need to be concerned about in your context. And your context is very important. Some code inspections are universal, where-as some are very context-specific.

Let's take Resharper for an example. For most of the code inspection issues, it provides a "Why is Resharper Ultimate suggesting this?" context menu option. Selecting this menu option brings up a web page with an explanation of the rule and why it is important to consider. This page can also give you options for how to remove the issue or to disable the warning in your code.

I have been guilty of ignoring code inspection suggestions/warnings. And I've also somewhat blindly changed code based on code inspection warnings without fully understanding what they mean. So this 'better way' is not something that is a perfect solution either. But I feel it is a good way to start.

So look at each issue and do the necessary research to decide if it is something you should be concerned with in your context.

### Involve Everyone

The biggest problem with code inspections is how they are interpreted differently by members of a team. For the rules to be useful (and to add value to your coding efforts) everyone on the team must have a clear understanding of the rules and how to deal with them.

Let's say a coding inspection reveals a possible typo (spelling violation) with a property of a class. You look at the property name and decide it is not a true spelling error and you want the issue to be ignored. You have two choices:

- Add a comment to the code that will disable the issue `// ReSharper disable once IdentifierTypo`, or 
- Add the new spelling to the user dictionary shared by the team. 

It may be obvious to you which is the correct response. But it may not be to everyone on the team. So it is important to establish procedures (whether they be formal or informal) on how to deal with each code inspection issue. Having discussions during the Pull Request process can be one way of ensuring everyone is on the same page. But it can be more useful to have regular team meetings, say once a month, to review the rules and procedures as a group. 

### Your Mileage May Vary

I have often wondered if using code inspections was worth the effort. StyleCop was more of a consistency tool, in practice at least. It made everyone conform to a consistent set of coding standards. Code inspections, like those built into Visual Studio or enhanced with tools like Resharper, are more about improving code quality.

It can take a lot of effort (sometimes) to ensure your code has no violations. So it is natural to ask, is the effort worth it? I believe that overall, yes it is worth the effort to look at the violations and make good choices (as a team) about which ones are applicable in your context. But measuring the impact they have can be difficult. You may never know if adherence to the rules improves the code quality.

But don't let that discourage you from making the effort. The process of learning about the inspections can be a great way of learning about good coding practices and techniques. It can show you different ways of accomplishing tasks in your code. It can sometimes show you something about your coding language you didn't know was possible. All this is to say, that you can learn a lot from using code inspections, even if they may not apply to your current context. 

### Summary

Using code inspections is standard practice for C# programming due to them being built into Visual Studio and amplified by extensions lie Resharper. But don't just ignore them or blindly assume they all need to be 'fixed'. Learn about each one and decide if they apply to you or not. Use it as an opportunity to learn more about good practices for improved code quality.
