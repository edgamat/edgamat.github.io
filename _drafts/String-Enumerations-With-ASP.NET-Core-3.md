---
layout: post
title: 'String Enumerations with ASP.NET Core 3'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
published: false
---
 
When building a Web API endpoint, you may need/wish to expose an enumerated value using
values that don't match the enumerated values in the code. The [DataContract] and
[EnumMember] attributes can help you achieve this.

<!--more-->

Let's say you have a property on a
