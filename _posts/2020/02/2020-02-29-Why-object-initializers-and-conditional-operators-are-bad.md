---
layout: post
title: 'Why object initializers and conditional operators are bad'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Originally published Nov 17, 2013.

<p>I recently listened to <a href="http://www.hanselminutes.com/396/bugs-considered-harmful-with-douglas-crockford" target="_blank" rel="noopener">a podcast from Scott Hanselman</a> where Douglas Crockford discussed ideas about good parts and bad parts of programming languages. I was familiar with Crockford’s wonderful book “<a href="http://www.amazon.com/gp/product/0596517742/" target="_blank" rel="noopener">JavaScript: The Good Parts</a>” and found the podcast to be equally enjoyable to consume. (If you haven’t already, I would recommend you listen to the podcast.)</p>
<p>Let’s face it, all programming languages are made up of good parts and bad parts. One of Crockford’s great insights is that he (and the rest of us programmers) create better programs when only using the good parts of a programming language.</p>

<p>It got me thinking about the day-to-day coding I do using C#. C# is a pretty impressive programming language. But what are the good parts? And equally importantly, what are the bad (and truly awful) parts?</p>

<!--more-->

<p>I wish I could give some great advice about all the good parts in C#. Perhaps on day I'll have something to offer on that topic. Today, I’d like to expose a couple of the bad parts. Namely, object initializers and conditional operators. Object initializers were introduced in C# 3.0 (Visual Studio 2008) while the conditional operator has been around since C# 2.0 (Visual Studio 2005). I have used these parts of the C# language for a long time and thought of them as useful tools. However, I now realize and believe that they are both bad parts of the language and should be avoided, if not eliminated outright.</p>
<p>Object initializers let you assign values to objects when they are created. For example:</p>

```csharp
person = new Person()
{
    FirstName = "John",
    LastName = "Doe"
};
```

Pretty harmless, yes? I’ve used this for years too and didn’t think much about it. But then more and more I saw it being misused. And the misuse was not subtle. It was tragic. A developer gets used to the object initializer syntax and soon forgets that there are other ways to initialize objects. 

And then this happens:

```csharp
var dto = new CreateAccountDTO()
{
    FirstName = person.FirstName,
    LastName = person.LastName,
    EmailAddress = person.EmailAddress,
    MailingAddressLine1 = person.Address != null ? person.Address.Line1 : string.Empty,
    MailingAddressLine2 = person.Address != null ? person.Address.Line2 : string.Empty,
    MailingAddressCity = person.Address != null ? person.Address.City : string.Empty,
    MailingAddressState = person.Address != null ? person.Address.State : string.Empty,
    MailingAddressZIP = person.Address != null ? person.Address.ZIP : string.Empty
};
```
<p>One must assume that the developer realized that the Address property could be null and would cause a runtime exception if it wasn’t checked. I offer that if the object initializer syntax was not used, the code is easier to read and easier to maintain.</p>

```csharp
dto = new CreateAccountDTO();
dto.FirstName = person.FirstName;
dto.LastName = person.LastName;
dto.EmailAddress = person.EmailAddress;
if (person.Address != null)
{
    dto.MailingAddressLine1 = person.Address.Line1;
    dto.MailingAddressLine2 = person.Address.Line2;
    dto.MailingAddressCity = person.Address.City;
    dto.MailingAddressState = person.Address.State;
    dto.MailingAddressZIP = person.Address.ZIP;
}
```
<p>To further my case against the use of object initializers, let’s look at the debugging and exception handling experience. When a runtime exception is raised, the line number returned in the stack trace is the line associated with the ‘new’ statement, not the actual line where the exception took place. For example:</p>

```csharp
try
{
    var person = new Person()
    {
        FirstName = null,
        LastName = null
    };
    
    var dto = new CreateAccountDTO()
    {
        FirstName = person.FirstName.Trim(), // exception occurs here
        LastName = person.LastName.Trim()
    };
}
catch (Exception ex)
{
    Console.WriteLine(ex.ToString());
}
```

<p>Line 11 is where the exception occurs, but because the code is using the object initializer syntax, the exception reports the exception occurring on Line 9. In other words, if a run-time exception occurs during the object initialization, you will never be able to tell which property assignment caused the exception. This alone is what made me stop using object initializers. I wasted too much time trying to track down the cause of exceptions. As soon as I switched to 'traditional' property assignments, tracking down exceptions became straightforward again.</p>

<p>Now I know, some might argue that NEVER using this part of the language is overkill. Programmers will claim that the problems I have sighted are due to inexperienced or incompetent programmers. They will claim they have the RIGHT to use this part of the language because they know how to use it responsibly. Bull. Programmers are bound (or ought to be bound) to create programs that are correct and easy to maintain. Our personal 'rights' are a distant priority. And if a part of the language is more likely to make the code harder to maintain and contain bugs, we have the RESPONSIBILITY to exercise good judgement and refrain from its use.</p>

<p>With that in mind, let's take a look at the <a href="http://msdn.microsoft.com/en-us/library/ty67wk28.aspx" target="_blank" rel="noopener">conditional operator</a>. It has been around since 2005 and I'm sure every programmer has found a use for it at one time or another. It allows for a compact conditional test that results in one of two return values:</p>

```csharp
var status = record.IsLocked ? "Locked" : "Unlocked";
```

<p>As with the use of the object initializers, this seems harmless and we've all used this language feature thinking it was a useful tool. But it is so easy to abuse or make code hard to read and understand. Here is one case in point. I have seen this done over and over by more than one programmer:</p>

```csharp
var receiveAlerts = string.IsNullOrEmpty(person.CellPhoneNumber) == false ? true : false;
```

<p>Unfortunately, the conditional operator was never conceived for this purpose. It is completely unnecessary to use this syntax, if the two expressions that follow the ? are boolean values. You can simply do this:</p>

```csharp
var receiveAlerts = string.IsNullOrEmpty(person.CellPhoneNumber) == false;
```

<p>Now, let's see what happens when checking for null values comes into play:</p>

```csharp
// person.ReceiveNewsletter is defined as Nullable
var receiveNewsletter = person.ReceiveNewsletter.HasValue == true ? person.ReceiveNewsletter.Value : false;
```

<p>Correctly using nullable types in C# is the topic for another post (because believe you me, it is totally mis-understood by almost every C# programmer I've come across). That being said, C# (since 2005 in C# 2.0) has had the ?? operator for dealing with these cases much more clearly:</p>

```csharp
var receiveNewsletter = person.ReceiveNewsletter ?? false;
```

<p>So, am I advocating that this sometimes-useful-often-abused feature of the C# language be scrapped? You better believe it. I'll admit, I've used this feature and thought it was perfectly acceptable. But now I feel differently. Too many times have I seen it be more trouble that it is worth. So yes, I no longer use this syntax. When I get the urge, I take a good look at the context I'm in and find another way to express my thoughts in the code.</p>

<p>Because that is who it really benefits; us, the humans reading the code. The compiler doesn't care. And it might take me 60 seconds (or even 60 minutes) to come up with a means to avoid the conditional operator. That's okay, because it is always better in the end. The codes is easier to read and is much less likely to contain errors. Using the example above of converting a bool (IsLocked) to a string, the following alternative is simply easier to read the intent of the code (and only requires you to write a reusable function once to gain this important benefit):</p>

```csharp
var status = BooleanToString(record.IsLocked, "Locked", "Unlocked");
```

<p>Another example of misuse I have seen over and over is dealing with default values. Typically, it involves reading settings (stored as strings) and converting them to integers.</p>

```csharp
string settingMaxAttempts = ConfigurationManager.AppSettings["MaxAttempts"];
int maxAttempts = string.IsNullOrEmpty(settingMaxAttempts) == false ? int.Parse(settingMaxAttempts) : 5;
```

<p>Actually, what I usually see is something much, much worse:</p>

```csharp
int maxAttempts = string.IsNullOrEmpty(ConfigurationManager.AppSettings["MaxAttempts"]) == false ? int.Parse(ConfigurationManager.AppSettings["MaxAttempts"]) : 5;
int timeoutMinutes = string.IsNullOrEmpty(ConfigurationManager.AppSettings["TimeoutMinutes"]) == false ? int.Parse(ConfigurationManager.AppSettings["TimeoutMinutes"]) : 60;
```

<p>Seriously. Who can think that this is a good idea? You ought to be making the code clearer to the reader. The conditional operator was introduced to reduce the number of lines of code a programmer would need to use in order to express a conditional conversion. Unfortunately, it has now devolved into a tool that allows programmers to force too much into a single statement. What we need to do is stop using this operator and create better code!</p>

```csharp
int maxAttempts = SettingsHelper.GetInt32("MaxAttempts", 5);
int timeoutMinutes = SettingsHelper.GetInt32("TimeoutMinutes", 60);

public int GetInt32(string key, int defaultValue)
{
    string settingValue = ConfigurationManager.AppSettings[key];
    int returnValue;
    if (int.TryParse(settingValue, out returnValue))
    {
        return returnValue;
    }
    else
    {
        return defaultValue;
    }
}
```

<p>I am not going to apologize for my solution requiring more lines of code to implement. It's easier to read and if a problem occurs, it is much easier to diagnose the issue. It's simply the right thing to do.</p>

<p>So there you have it. The object initializers and conditional operators are not good parts of C#. We shouldn't use them. We have better alternatives at our disposal to create programs that are easier to read and maintain and contain fewer bugs. Shouldn't that we our priority, rather than the reduced number of keystrokes we are required to use? The premise that fewer keystrokes makes us more productive has proven to be false. When we use these 'shortcuts' in the syntax, the code is more apt to be of poor quality.</p>

<p>We are simply more productive when we do our jobs better and take more responsibility for the long-term quality of the code.</p>
