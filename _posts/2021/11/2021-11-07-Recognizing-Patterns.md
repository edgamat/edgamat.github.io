---
layout: post
title: 'Recognizing Patterns'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I was called in on the weekend to diagnose an issue with an application I wrote. I was very lucky to spot something in the application
logs that led to correcting the issue. Recognizing patterns is pretty important in my job.

<!--more-->

### The Background

On a Saturday morning, my manager texted me letting me know that a job we had kicked off the night before was running slower than expected. We estimated it would process over 1,000 messages per minute, and it was only managing 500 per _hour_. Something was up, and he asked me to take a look.

He told me how he was calculating the metrics, which was on the producer side of the message processing. It just wasn't publishing messages fast enough. that was a new one for me. Normally when a slow down is reported, it is on the consumer side. 

So I looked at the code to see what possible causes their could be. The message producer was reading a 'queue' table to get work to be processed. Each record in the queue represented a new message that needed to be produced. When it was done producing the message, it update the record in the table as 'processed', removing it from the queue. In addition, the code that actually produced the message, read data from a different database server. 

So I had several avenues of inquiry at my disposal:
- Maybe there was a problem reading/writing from the table, or
- Maybe the code that produced the message was slow (ie the message broker), or
- Maybe the code used to produce the message was slow.

On a Saturday afternoon I was looking for a quick fix, and none of these looked like they would be easy to diagnose and fix.

### Looking for Patterns

In the queue table, the application logs a timestamp each time the status is changed. So when a record is marked as 'processed', the code updates the timestamp to the current time. I decided to look at the database to see what some of the timestamps looked like. 

It could be possible that the slow downs were random, or their could be a pattern. Perhaps the slow downs all took place near the top of the hour (something that has occurred in the past). Perhaps one or two messages got 'stuck' and that was slowing down all the rest. 

When I started looking for these patterns in the logs I noticed two things. First, there were a lot of messages processed one after the other rather quickly (less than 1 second per message). But I also saw a lot of messages that were taking 10 seconds to process. Those would definitely be contributing to the slow rate of producing messages.

But why 10 seconds? That was odd. If there were problems reading or writing data to the databases, it is very unlikely to see such a regular pattern. What could cause that pattern?

### A-Ha!

I looked in the code to see what was being done when looping from one message to the next. The loop included logic to pause processing when no more message were in the queue. It would then periodically poll for new messages. This reduced the load on the server when there is no work being processed (the majority of the time, this is the steady state). And sure enough, the interval between each poll of the queue was configured to 10 seconds.

So my assumption was the system believed there was no more work to process in the queue and it was waiting 10 seconds before processing the next message. The source of the problem was how the code determined if no more work to perform. And once I knew that, I went about coding a fix. A few unit tests later and I had a pull request ready for review. 

The code change ended up fixing the issue and we saw the rates increase to 1,000 - 2,000 messages per minute, just as we had expected.

### Recognizing Patterns

I think diagnosing problems with a lot of things boils down to recognizing patterns. It's something that comes naturally to me, but I have worked hard to improve that skill. It involves a lot of training, and a lot of experience. The more problems you diagnose, the more patterns you have in your toolbox to look for. I would encourage any developer, regardless of their experience to look for ways to improve their pattern recognition skills. It has saved me many times from having to spend weekends fixing bugs :-) 
