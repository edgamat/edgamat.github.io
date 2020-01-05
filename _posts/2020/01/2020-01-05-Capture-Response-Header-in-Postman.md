---
layout: post
title: 'Capture Response Header in Postman'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

When building a Web API endpoint, my team uses [Postman][postman] to send requests to the endpoints
and confirm the correct results. One could argue that Postman is our user interface. One use case
we run dozens of times a day requires us to copy-and-paste a value from one response into the request
for another endpoint. It is slow and tiresome and I was curious if Postman could help us be more
efficient. Turns out, I was right.

<!--more-->

## Our problem

One of our endpoints accepts a JSON payload for processing. It returns a 202 CREATED response, and
includes the URL where the status of the processing can be checked. The URL is returned in the `Location`
header of the response.

![sample showing the location header](/assets/img/postman-location.png)

We then copy-and-paste this value into another tab in Postman and run a GET request to check the status.
This step of copy-and-paste is what I'd like to eliminate.

## The Tests Tab

In Postman, there are a series of tabs where you can modify the settings for each request:

![sample showing the tabs](/assets/img/postman-tabs.png)

Mostly, we set the authentication and body of requests, but there is also one there labeled `Tests`. This
feature allows you to write scripts (in JavaScript) and run them against the response, after it runs.

More info: <a href="https://learning.getpostman.com/docs/postman/scripts/test-scripts/" target="_blank">
https://learning.getpostman.com/docs/postman/scripts/test-scripts/
</a>

In the editor, I wrote the following:

```javascript
pm.environment.set('lastStatusUrl', postman.getResponseHeader('Location'));
```

As it implies, it sets the `lastStatusUrl` variable in the current environment to the
value returned in the `Location` header.

Then I created a new GET request that uses this variable to execute the request.

![sample showing the get last status request](/assets/img/postman-last-status.png)

Now whenever I run the first request, I can run the Last Request Status request which will have the
URL already set and ready to go. No copy-and-paste required.

## More to Discover

I am very new to Postman so I have a lot more to discover. But finding the ability to run scripts
after a response is available would seem to open a new world of possibilities.

[postman]: https://www.getpostman.com/
