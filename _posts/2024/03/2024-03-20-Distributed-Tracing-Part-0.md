---
layout: post
title: 'Distributed Tracing in .NET - Part 0'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

I want to explore distributed tracing in .NET. For that I'm going to need a sample application to instrument.

<!--more-->

### The Problem

I one worked on a team that built out a series of microservices using Rabbit MQ, SQL Server and ASP.NET. One of our biggest issues was our inability to
obverse requests as they went from one service to the next. We couldn't easily debug issues in production because we couldn't see what was happening
in our applications. We used structured logs, but each service contained its own logs. Correlating requests between services was next to impossible.

What I'd like to do is explore distributed tracing as a means of helping gain visibility into requests as they travel through the resources in a microservices
architecture. 

### Project "Ambition"

I am creating a sample application called "Ambition". It is a reference architecture demonstrating a simple set of services working together.

Here is the 'lore' of the application:

> Our business revolves around selling maintenance plans for widgets. Our customers have widgets that they want to be serviced on an annual basis. Our sales people work with these customers to determine the best maintenance plan for their widgets and then use our software to record the details of the plan, send payment terms to accounting, and send an email to the customer.

Each plan is associated with a product id and a customer id. Each plan has an associated cost, effective date and a description of the widget.

There is an ASP.NET Web API where the system sends requests to create new plans.

The CREATE PLAN endpoint creates a new maintenance plan in a SQL Server database table, and then publishes a "MaintenancePlanCreated" message to RabbitMQ.

The Accounting Service consumes the "MaintenancePlanCreated" messages, recording the plan in its database setting up an invoice to be paid. It calls
a third-party HTTP API that sends an email message to the customer.

The .NET solution includes:

- Ambition.UI - the Web API project used to record new maintenance plans
- Ambition.Domain - the domain objects for the UI
- Ambition.Infrastructure - a set of services to interact with databases, message queues, etc.
- Ambition.Accounting - the accounting service
- Mercury.Email - a Web API project that acts as a fake email service

There will be 2 databases created:

- AmbitionSales
- AmbitionAccounting

### My Goals

I want to send a request to the CREATE PLAN endpoint for a given product and customer. Each request is sent by a user (identified by a user name).

I want to trace the request through the system, being able to see:
- that it got created in the database (creating a maintenance plan id)
- that a message was sent to the accounting service
- that a new invoice was created 
- that an email message request was sent to the third-party email provider 

I am going to use the `System.Diagnostics.ActivitySource` to create tracing data.

I am going to use Open Telemetry to export the telemetry data. 

Initially I am going to use Seq to record all the tracing data. Using Seq, I should be able to filter the logs and view traces of the steps along the way.

I want to inject errors at various points and see what the tooling provides me.

I want to inject slow downs to see if the tooling makes it possible to diagnose the issue.

The source code can be found here:

[https://github.com/edgamat/Ambition](https://github.com/edgamat/Ambition)
