---
layout: post
title: 'Clarity Over Cleverness'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---
 
I read a tweet this week that made me think. It was from [Seth Juarez](https://twitter.com/sethjuarez), a manager at Microsoft.

> Most of our problems in tech stem from our desire to be clever as opposed to simply being clear. Cleverness over clarity is our Achilles heel

<!--more-->

![Most of our problems in tech stem from our desire to be clever as opposed to simply being clear. Cleverness over clarity is our Achilles heel](/assets/img/Seth-Tweet.PNG)

### What's Wrong with Clever Code?

In my experience, 'clever code' is a solution to a problem that is not obvious to the expected audience of the code. And in the vast majority of the cases, the expected audience is not you. This implies you should be writing code to be clear to others, not yourself.

It is very natural to want to code something as it appears to you in your mind. You start out writing the code and before you know it you have a working solution just as you had imagined it. But then you have to explain it to someone else and that might be more difficult than it seems. 

I have worked with many smart people in my career. A lot of them have given me feedback on my code that improved things by making the code clearer. I am not saying 'simpler', but rather 'clearer'. What's the difference?

### Clear Code

Some people (especially folks influenced by the culture surrounding Apple products) feel that simplicity is the answer to great design. Extending that into the software world would imply that simpler solutions are what we should be striving for. I won't debate that concept here, but I want to differentiate between simple and clear.

Code is 'clear', in my humble opinion, if it can be read and understood on its own, without any supporting material or explanation. Sometimes you might need to provide readers with some context and that is expected. For example, you might be more familiar with a set of business concepts than a co-worker, or perhaps you are an expert using a particular tool or framework. Providing this context for readers is important as well as filling in any gaps in someone's knowledge. But you should avoid using 'jargon' that might confuse readers. 

### The Three C's

Early in my career someone gave me some notes on how to be a better writer. While I am still struggling with that (as this blog demonstrates), one approach they advocated was writing using the "Three C's":

- Clear
- Concise
- Consistent

I find these are all admirable traits in the code I write as well. 

**Strive for Clarity** 

When I prepare my code for a pull request (PR) I create a draft for the PR that shows me all of my changes. I will typically find that some of the code changes aren't clear. A method or property might not have a good name, or a class might be too long. Or it may not be clear how the code satisfies a business requirement. I spend the time to improve the situation before sending the PR to my co-workers.

I use several techniques to achieve clarity including refactoring the code. But the two most frequent techniques I use are to move code and to split up code.

Moving code helps ensure each class or function strives to do one thing, in a cohesive way. I'll find that I have too much code in a controller action method so I move it to a domain or helper class. I'll move data access logic out of a domain method into something related to persistence. 

When I write the code (the first pass) I'll just be glad to get it all written and working. But then I take the time to move the code around so it is better organized. I also will split code apart. More often than not the methods I write are too long or hard to explain without comments. I split the code into smaller functions and if I feel the urge to write a comment I try to see if the block of code would be better served by a separate function instead.

**Keep it Concise**

Sometimes I will take this one too far, but being concise is usually a good thing. Why write 10 lines of code when 4 will do? Sounds like good advice. But sometimes you can be too concise and that hurts the clarity of the code. I see a lot of folks get 'too clever' when trying to be concise. They 'force' something into as few lines as possible (too many C programmers fall into this trap). But I try to strike the right balance. I take out all the code that isn't necessary but don't sacrifice readability in the process.

**Be Consistent**

Once a codebase has grown to a sufficient size, most of the coding patterns are reusable throughout the code. New features that come along are not usually breaking new ground. You should strive to follow the existing patterns for the sake of consistency. Sometimes clear code can mean ensuring that the patterns in the code as easy to recognize. Why make it harder on the reader than is necessary? Coding standards are also important for achieving this part of clear code. Anytime you can present something the reader recognizes and is used to seeing the clearer the code tends to be. 

### Summary

I strive to write clear code. Clever code can cause others on your team to mis-understand what the code does so it is important to use techniaues that result in code that is clear, concise and consistent. 

NOTE: Inspiration for this post came from the following source:

[Clear, concise, consistent – The three Cs of effective communication](https://www.masongroup.ca/blog/clear-concise-consistent-effective-communication/)


