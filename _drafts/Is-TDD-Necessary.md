---
layout: post
title: 'Test-Driven Development: Is it necessary?'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

This is a post originally written in May of 2014. Still relevant today I feel. Not my best writing to be sure, but I love the Google+ reference.

<hr/>
<!--more-->

<p>Over the past week or so, I have been watching an interesting podcast (excuse me, Google+ ‘hangout’). It is a series of conversations discussing test-driven development (TDD).</p>

<p>It gave me a lot to consider. I have never been able, nor found it necessary, to use a super diligent form of TDD on any project I've worked on. I have however, used TDD in situations where I thought it would be useful. And I am a fan of TDD. I like the experience it provides. It gives me the feedback I am looking for and I always feel that the solution I have come up with is a better design.</p>

<p>So, does it then follow that I should be using TDD for everything I do? I think that is the question the podcasts I have been watching are dealing with. The video (and audio) are available here:</p>
<p><a href="http://martinfowler.com/articles/is-tdd-dead/" target="_blank" rel="noopener">http://martinfowler.com/articles/is-tdd-dead/</a></p>

<p>I listen to the argument for TDD and against. I agree with both sides; well, in certain situations. So I am not 100% in either camp. I have a great deal of respect for the folks giving their opinions.  And most of the comments other listeners have provided are pretty respectful. Very few are of the 'snarky-bitter-troll-comment' variety. Most are pretty reasonable.</p>

<p>So how do I use TDD? Why don’t I use it all the time, for everything I do? I think back to late 1999 when I first became aware of eXtreme Programming (XP). It was a new way of thinking of software development. I read the books, I even tried some of practices (unit testing, pair programming, continuous integration, refactoring, and others). Some of these practices I use very often (daily) and I feel they make my code of better quality. But, as with TDD, I don’t use them ‘to the extreme’, all the time. I use these practices where I feel they will do the most good.</p>

<p><strong>Coding Is a Risk Management Strategy</strong></p>

<p>For me, coding is a risk management strategy. I look for risk in the code I write and I calculate (well, it is more like a gut feeling) the probability of the risk occurring. And, just as importantly, what the impact of the risk might be to the project. I have to consider a variety of factors, including some that are very subjective. But I look at these factors and use everything at my disposal to reasonably manage the risk.</p>

<p>So I look at the code I need to write and perform a little planning cycle in my brain (or on paper). Is this something I really know how to code? Or is it something I will only really know how to code AFTER I have written it (those darn <a href="http://en.wikipedia.org/wiki/Wicked_problem" target="_blank" rel="noopener">Wicked Problems</a>!). The more I don’t know how to code something, the more likely I am to use TDD. But this isn't the lion’s share of the code I write. Most of it is pretty straightforward. Get data in and out of a database using a web application. Run reports on the data. Repeat. I enjoy helping people solve their business problem using technology. It doesn't all have to be cutting-edge stuff. Companies can get a lot of business value from just having access to the data, organizing it, and making sense of it all.</p>

<p>So, is TDD ‘necessary’ in all coding endeavors? No, I’m afraid I don’t think so. I think it is a very good technique to know and do well. And it should almost always be used when the risk of not doing it is enough to cause your client grief. Of course, there are those who will say the risk is ALWAYS there. Therefore you should always use TDD. And that is okay for some people. It just isn't for me.</p>

<p><strong>What Does TDD Buy You?</strong></p>

<p>Part of the risk assessment of using TDD or not must consider the amount of time spent fixing errors in the code (or in the design) late in a project. TDD has a better probability to find these errors earlier in the process. At least, that is what some people argue. And I have definitely seen where that is the case. This is one of the reasons I use TDD. I want to really be sure the design of the code is correct, and TDD gives me the best chance at getting it right.</p>

<p>However, the use of TDD doesn't necessarily reduce the amount of actual time spent fixing errors. In fact, I have seen the TDD anti pattern of ‘false sense of security’ appear a lot with developers trying to use TDD. People write test cases, then the code to make them pass. And since the test runner shows all green, everything is okay! Unfortunately, they spend just as much time fixing bugs as anyone else.</p>

<p>The simple truth is that there is more to TDD than <a href="http://www.butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd" target="_blank" rel="noopener">Uncle Bob’s ‘Three Laws of TDD’</a>. Writing the tests first, and then the code to make the tests pass is only a part of what really makes TDD a good practice. It also requires the ability to write good tests! One of the core pieces of test-first practices is to test everything that can break. That aspect is often overlooked. If you aren't already good at testing, TDD is not going to make you better. In fact, it probably will make you worse off. because you will (falsely) believe your code is free of bugs. TDD should be where you want to get, not where you should start. Learn how to determine what to test, learn about white-box testing and how to choose tests that will have a high probability of finding bugs. Then try to automate the tests (no point in automating crap... only worry about automating things once they are actually useful). If you can automate the tests, then try and make them repeatable. Once you've got a good handle on this, then, you might be ready to take on TDD.</p>

<p><strong>TDD Can Be Done Wrong (Just Like Anything Else)</strong></p>

<p>Recently, I had to review some code that another developer had designed using TDD. Disappointingly, it was riddled with bugs. It simply didn't work, not even the simplest real-world use case would run successfully. But, it had dozens of automated test cases (written in a TDD methodology) and if you ran them all, you got all green from the test runner!!! So what was the problem? Why did the code not really work if the TDD methodology was used? Great question, I think. Was it due to TDD being done poorly? Perhaps. Nobody’s perfect. But I felt after speaking with the person who developed the code, that their understanding of TDD was very good. This was a smart, hard-working developer. So why did the code contain so many problems?</p>

<p>In the end, I (subjectively) felt it was due to a lack of good <span style="text-decoration: underline;">test analysis</span> skills. The developer hadn't thought of the right things to test. Well, some of right things were there. But not most of them. He hadn't fully inspected the results being produced by the code to see the errors. How did I spot them? With my eyes. No TDD automated test is going to catch everything, granted. But my experiences with analysis and creating test cases had taught me what to look for. And I didn't need automated test cases. I’m speaking of good old fashion eyes and brains.</p>

<p>A developer who wishes to used TDD must be able to use their skills to analyze the problem and figure out how to prove the results are correct. Proving the results are correct is difficult. Or time consuming. Or both, if you are unlucky. But you need to be able to write down on a sheet of paper (or tablet, or whiteboard) what you need to be looking for when proving the code does what you think it should. I humbly submit that whether you automate a test or can run it repetitively is not as important as whether it is actually a good test to use in the first place.</p>
<p><strong>Is TDD the Reason the Code is Better?</strong></p>
<p>I wonder. Do people who use TDD effectively do so because of their good analytic skills? Would these people produce good code even if they didn't use TDD? Is TDD simply a more effective tool for those already predisposed to writing better code? It would explain a lot of my observed effectiveness of TDD. Having poor testers use TDD doesn't make their code better. It just takes them longer to write bad code. Okay, maybe that is mean, but in some cases, it's the observed result.</p>

<p>I use TDD when it make sense. I don't use it always. Does that make me less likely to produce good code? I don't know. I am always learning new things and tomorrow I may learn something that makes TDD the greatest thing in the world. But for today, it is just one of the good tools I know about and use.</p>

<p><strong>Remember:</strong> For every job there is the perfect tool. I have yet to find a tool that is perfect for every job.</p>
