---
layout: post 
title: 'Prototype Versus Mockup'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
published: false
---
 
Some people use the terms prototype and mockup interchangeably. In my mind, these are two different concepts and serve slightly different needs. 
 
<!--more-->

### What is a Mockup?

I define a mockup as a representation of what you want to achieve. It is a means of explaining ideas, discussing concepts and requirements, and gathering feedback.

In the world of software development, I feel that a mockup is a static or interactive representation of the software. Let's say we are discussing a web application. The mockup might be a series of web pages designs. The pages may be interactive, but that is not necessary. A non-interactive mockup may be called a 'wire frame'.

### What is a Prototype?

A prototype is a working piece of software. A prototype is built as a means of learning something about the final product you are trying to build. You may want to learn about the challenges involved, in order to come up with planning estimates. Or you may want to learn about how to use a tool, or set of tools, to create the software. Or you may want to create a guide for others to use when developing the 'real' application.

A vertical prototype is one where you build out a portion of the application, through all the layers. For example, you may build a web page that calls a backend server which in turn calls a database to manipulate the data. This style or prototype can be helpful in defining the interfaces between the layers and the way your architecture may be developed.

A horizontal prototype is one where you build out a thin slice in one layer of the application. This typically means building out a series of web pages that demonstrate the user experience and behavior. It can really help define the cope of the UI work necessary to build out the solution. 

### When to use one or the other (or both)?

I recommend using both. When using an agile system to manage your project, it can be very useful to have a mockup when discussing a product backlog, creating stories, grooming stories and prioritizing the items to be worked.

A prototype is more useful to plan out a sprint or breaking down a story into tasks for developers to work. 

A mockup and a prototype have a lot of similarities, but some important differences. A mockup can be created by a business analyst or stakeholder. I've seen them created using simple HTML, tools like Invision and Balsamiq, or PowerPoint presentations. I have seen a mockup created on a whiteboard in a joint application development session. Mockups are easy to change and rework, which make them a small investment to make and gain a lot of benefit for your project planning.

A prototype on the other hand, is a higher level of effort. A developer typically is creating a working piece of software using the tools you intend to use to build the actual application. Sometimes a 'throwaway' prototype makes sense. You create the prototype with the intention of learning from the experience, but not actually using the code. But you can also use an 'evolutionary' approach and the prototype evolves into the final application. 

But this additional effort for a prototype can be very beneficial. Developers can discover implementation issues earlier in the process. And a prototype can help understand exactly what components need to be build by the team.

### Summary

I have tried to define my ideas of a mockup and a prototype and how to user them. Each has a distinct purpose and used a different phases of a projects lifecycle. Use both to get the most benefit of these important tools.
