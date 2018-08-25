---
title: Transactions and Failover using Saga Pattern in Microservices Architecture
layout: post
date: 2018-08-24 22:48
image: "/assets/images/profile.jpg"
tag:
- microservices
- architecture
- saga
category: blog
author: ahmedsaid
description: An introduction in using Saga Pattren for handling the transaction
  operations in Microservice architecture world.
---

## Summary:

In this article, I'll introduce to you the Saga Pattern for distributed transactions and show up how it can help in building robust business transactions flow in microservices architecture.

### Introduction

I'm sure you have heard about **Two-Phase Commit** , It's very popular approach to build transaction operations which is in summary when the commit of a first transaction depends on the completion of a second. It's very straight forward and easy specially when it comes to update multuple entities at the same time like confirm in order and update the stock at the same time.

But, when it comes to **Microservices**  things became complicated as most of application's parts are distrubted among diffrent services and every service whith it's own data storage, and you no longer can leverage the simplicity of local two-phase-commits to maintain the consistency of your whole system.

Let's take this example, say we're building *Travel & Booking Website*, and we started with very simple architecture.

![booking-architecture](/data/booking-architecture .png)

>In the example above, one can't just place an order, charge the customer, confirm booking with supplier, and send confirmation email/SMS to customer all in a single **ACID transaction**. To execute this entire flow consistently, we would be required to create a distributed transaction.

### Problem

Building distributed transaction across multiple services counted as very complex and tricky task as we have to consider many issues may take place like dealing with service availabiity with transient states, eventual consistency between services, isolations, and rollbacks all these scenarios  should be considered during the design phase carrefully.

### Solution : The Saga Pattern

A Saga is a sequence of  transactions where each transaction interacts with its corresponding  single service. The first transaction is initiated by an external request corresponding to the system operation, and then each subsequent step is triggered by the completion of the previous one and it contains the mechanism of handling rollback for the whole transaction sequence.

Using our previous  example, in a helicopter view, a Saga implementation would look like the following:
![booking-flow](/data/booking-flow.png)


There are a couple of different ways to implement a saga transaction, but the two most popular are:

- **Command/Orchestration:** There's an orchestrator which responsible for centralizing the saga's decision making and sequencing business logic.

- **Events/Choreography:** There's no coordination or orchestrator, each service integrates and listens to the other service's events and decides if an action should be taken or not.

#### Command/Orchestration is my favourite one for these reasons:

- Avoid messy dependencies between services, as the saga orchestrator the one who invokes the saga participants.
- Reduce  complexity as they only need to execute/reply commands.
- Easier to be implemented and tested.
- The transaction complexity remains linear when new steps are added.
- Rollbacks are easier to manage.

### Let's dive more

In the orchestration approach, we'll create a new service which will left the responsibiity of telling each participant what to do and when. The saga orchestrator communicates with each service in a command/reply style telling them what operation should be performed and will take the responsibility of firing rollbacks if needed.

![saga](/data/saga.png)

1. Order Service saves a pending order and asks *Order Saga Orchestrator* to start a create order transaction.
2. Orchestrator  sends an execute payment command to *Payment Service* and await for feedback on orchestrator queue channel.
3. Orchestrator sends a confirm booking command to *Booking Service*, and await for feedback on orchestrator queue channel.
4. Orchestrator sends execute send notification to *Notification Service*.
5. Orchestrator execute confirm order in *Order Service*.

In the case above, Order Saga Orchestrator knows what is the flow needed to execute a "create order" transaction. If anything fails, it is also responsible for coordinating the rollback by sending commands to each participant to undo the previous operation.

> *Notice:*  We should move the operations which we cannnot rollback to the last transations like Notifications/SMS as we can not revert sending SMS sending operation etc.

### Rolling Back in Saga's Command/Orchestration

Rollbacks are a lot easier now, the Orchestrator should fire execute compansation/rollback event once needed to the corresponding service.

Example: If the booking has been failed from the supplier for any reason after we take money from the client for any reason, we should refund the money to the customer again.

![rollback](/data/rollback.png)

However, this approach still has some drawbacks, one of them is the risk of concentrating too much logic in the orchestrator and ending up with an architecture where the smart orchestrator tells dumb services what to do.

>I'm hearing you know asking about **Events/Choreography** the second approach of Saga Pattren,  I'll talk about  in the next article ASAP.

---

Hope you enjoyed it, If you have any questions, feel free to ask me at comments section below.
![bye](https://media.giphy.com/media/dnQ4wtRVD0sTK/giphy.gif)