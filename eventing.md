---
title:  On Availability & Event Sourcing 
summary:  Introduction to Enterprise Events
author: Raja Shankar Kolluru
permalink:  events.html
carousel1: images/ext/CQRS.png
carousel2: images/ext/CQRS.png
carousel3: images/ext/book-stack.jpg
featured: images/ext/event.jpg
tags: Events  DDD ThinEvents ThickEvents Messages RPC APIDesign CQRS Availability EventSourcing
quote: It had long since come to my attention that people of accomplishment rarely sat back and let things happen to them. They went out and happened to things.
quoteAuthor:  Leonardo Da Vinci
---
## The Context
We will start this post with a story of a big bad launch of software. There was a massive corporation that decided to embark on a project of breaking a massive monolithic piece of software into a million chunks of smaller software - well not literally million but quite a lot of chunks.  

There were a lot of teams created - each responsible for their own piece of software. They developed their individual pieces, agreed on contracts amongst themselves, tested the pieces individually, assembled them together and waited with bated breath as they switched the integrated system to start. Nothing happened.  The damn thing did not come up. 

Four nights of debugging and several coffees later, it was realized that there are too many nested calls between micro services. One micro service called another which called a third one and so on till the call stack started looking more like a tower of Babel. 

The biggest victim of this kind of indiscriminate call stacks is availability. As the call stack gets more and more nested, availability decreases dramatically. 
To explore availability further, let us explore a typical situation in an enterprise application - wherein a client is trying to obtain some data from a server.

## Client-Server Interaction
In this case, the server provides the data which the client needs. Server also has the proper way of retrieving the data in the way the client needs i.e. the data retrieval process is tied to the client. For example, let us consider a product catalog in a retail enterprise. The search app requires this to be indexed by various product descriptions while the pricing app requires a  different way of retrieving of the exact same data. 

The typical client-server interaction model is a one-to-one request-response communication protocol. The client issues a request to which the server responds. The server is customized to support various ways in retrieving the data.

Two important paradigms within the client-server interaction model are RPC and Messaging.
### RPC Calls
 Remote Procedure Call (RPC)  operates on a synchronous request-response paradigm. RPC clients and server are temporally connected i.e. both need to be up and running at the time of making the call. If the server were down, the client will need some kind of circuit breaker pattern to retry till such time the server is available.

###  Messaging
In this paradigm, a highly available message broker  intermediates interactions between the client and the server thereby breaking the need for the temporal relationship between them.  However this still does not mitigate the relationship between the client and the server. 
The client  processing is still tied to receiving a response from the server.   If the server is unavailable for 20 minutes, the product catalog, for all intents and purposes,  is unavailable for the same amount of time. So the availability problem persists - albeit without the need for circuit breakers and the like. 

## Bringing Data Closer to the Client
The best way to break the availability problem is to create a redundant source of data.   The redundant data source services the client even if the server is available. This hugely increases the availability and in the process buys us considerable freedom. We can structure the data as per the retrieval needs of the client. So we not only address the availability problem but have considerably increased the performance of the application. 

We can go crazy and push the data for every client in the enterprise thereby cluttering the enterprise with the same data everywhere. For example, we can create miniature replicas of the product catalog in every system in the enterprise. Alternately we can stipulate some designated sources for this data - anyone who needs the data must tap into one of these designated sources. 

If I design an enterprise service that requires the product catalog, then I will tap into the data source that supports the product catalog retrieval that is most suitable for me. This realization is documented in the CQRS pattern. The Command i.e. the write part is separated from the query i.e. the read part. All reads are serviced by the query DB whilst all the writes are serviced by the command.  There can be multiple query databases for the product catalog but there exists only one master source which is called the Command DB. The data diagram illustrates this pattern. But there is more. 

## Data Proximity
 Let us say I am automating the stores in a large enterprise. The stores must continue to work even if the connectivity between the store and the rest of the enterprise is down. Hence the product catalog must be replicated in every store. This adds a new dimension to the replication strategy. There must be as many data sources as there are stores!

 So the exact same replicated product catalog exists in multiple stores (as shown in the diagram)

## Events
How do we make sure that the data can propagate to the myriads of data sources within the enterprise? One way is to make sure that when the data is mutated within the master source then the entire enterprise gets the mutated data immediately. We can transactionally link the entire system to make sure that the data command only returns SUCCESS if all the replicated databases have successfully changed! This gives us data reliability at a huge expense - it sacrifices availability which was the original driver for this exercise to begin with!

Hence the industry has drifted towards a slightly less reliable but a much better option - events. Events are propagated for every data change. These events are used by the individual data bases to mutate themselves. So when a new product (in our example) gets introduced into the product catalog then the product is created by every replica of the product catalog. 

This gives us huge availability gains albeit at a slight loss of consistency - a phenomenon called eventual consistency. Eventually the DB becomes consistent. If I might be excused a pun: 
> Events make the DB consistent - eventually. 

## A Deeper Look at Events
Eventing has become mainstream with the huge success of KAFKA and Micro Services. We have witnessed a phenomenal increase in its adoption with multiple event sourcing strategies propounded for data propagation in the enterprise.

But eventing is not always easy to fathom. Canonical event design in a massive enterprise is a gargantuan task that cuts across organizational boundaries. It will involve multiple stakeholders, diverse technology stacks and the resolution of conflicting priorities.

We are in the midst of these exercises in multiple companies. Many a time, I see a lot of confusion about what constitutes an event. How is it different from a message?  To which my answer is that events can be messages but not all messages are events. 

If a large enterprise were to send ETL feeds everywhere, will it make sense to replace these feeds with events? Is it an incremental ETL strategy? What is the difference between a feed and an event? 

These are some of the questions that we will attempt to answer by stating some axioms of events:

## First Axiom - Event Represents Change
This might seem obvious but nevertheless bears stating. An event happens when there is a change in the system. There is no reason to emit an event if there is no change that has happened to the system. 

## Second Axiom - Event Represents an Axis of Change
 Let us consider our product catalog example. A product catalog has multiple products. with attributes like name, description etc. Products are also associated with prices. 
 
 However, price and other product attributes have different axis of change. Prices change more often for a product than other attributes.  The stakeholders for price change are different from the stakeholders for addition of new products or for product change. Hence in most retail enterprises, there exist two bounded contexts - product and price domains.. Each domain has its own axis of change. 

> An event represents one axis of change 
> Corollary 1: An event precisely represents the changes occuring for one entity in a bounded context.
> Corollary 2:  An event cannot be shared across bounded contexts.

### What is wrong if the event represents multiple axis of change? 
Typically, this will cut across multiple bounded contexts and has the potential to cause disruption since the ownership of the event gets diluted. In this example, we would need a consolidating bounded context which unifies both price and product into one entity. This is possible and sometimes desirable as well. But this axiom makes it clear that there is work that needs to be done in unifying the required bounded contexts and generating the event.

## Third Axiom - Event Size is proportional to the processing
Events are one time affairs and are mappable to different things that happen to particular entities. Views represent 

(to be continued)





