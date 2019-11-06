# Summary of a four days DDD training

> Make your domains shine!

I think that one could say I was already convinced by the benefits of the *Domain Driven Design*.

I learned the hard way that driving your software engineering by the technical end can lead to pretty catastrophic situations.

Technical debt, hard coupling with any data provider, having to translate every domain feature request to fit in the already infected code base... I found myself in this situation not long ago and it was really hard for me to find the root cause.

This is when I found out about DDD. It immediately connects to me as it was exactly the thing that I was not able to name! All of those smells could be avoided by working with the domain to create a software that really reflects it.

I started diving into the subject by reading *Eric Evans*' [**Blue Book**](https://www.oreilly.com/library/view/domain-driven-design-tackling/0321125215/) and *Vaughn Vernon*'s [**Green Book**](https://www.oreilly.com/library/view/domain-driven-design-distilled/9780134434964/) and we started implementing a few of the principles in my job, with the help of a coworker that already did in a previous mission.

I followed a four days (french) training given by [*HackYourJob*](https://twitter.com/hackyrjob)'s **[Florent Pellet](https://twitter.com/florentpellet)** on *DDD* and its implementation. It was an excellent experience as he was able to validate what I already learned while teaching me a whole lot of new concepts.

Here are my organized thoughts and notes about those great four days.

> Disclaimer: I wrote this article mainly to have a clean trace of this training. I would be glad if you find it interesting and if you use it as a quick reference, but I highly recommend that you dig deep into each subject.

## My own expectations

My main expectations for the whole training was the following:

- Validate what I previously learned;
- Learn how to introduce the subject and onboard a tech-oriented team;
- Here some real world stories about the usage of *DDD*;
- Have a clear definition of two *DDD* terms that I often mixed up: *Strategic* and *Tactical* patterns;
- Learn more about *DDD-lite*, is it a fraud?

## What is *DDD*?

We started the training with some theoretical definitions.

> Note: Every definition will be stated as blockquotes. They are rewording of concept that I learned about in the training and do not engage anyone but myself.

Obviously, it was a good idea to start with the explanation of *what is DDD*:

> *DDD (Domain Driven Design)*: A set of tools aiming to manage **software complexity**.

But what is *software complexity*? *Ben Moseley* and *Peter Marks* started their essay **Out of the tar pit** by stating that *"Complexity is the single major difficulty in the successful development of large-scale software systems"*. It can be divided into three types: 

> *Essential complexity*: The complexity that cannot be reduced, that is required to answer the need.

> *Mandatory accidental complexity*: The one can we cannot do without (the network, the programming language, etc).

> *Optional accidental complexity*: The complexity caused by the use of an inadequate framework, by the use of a different language between the domain concerns and the development one, etc.

The latest complexity type is the one that we are trying to reduce, while trying to manage the three of them better with *DDD*.

## It's all about **context**

Writing a piece of software can be reduced as finding the right abstraction to answer a need. This abstraction is deeply linked to the context of the need.

We are trying to answer a **specific need**, as opposed to a generic one.

Take this common example of a map. If you try to help a friend reaching his destination using the subway, you are not going to give him the entire map of the city. You will certainly give him the subway map, the *abstraction of the city in the context of the subway lines*.

> *Bounded context*: Set of elements (classes, services, etc) containing only what is needed in a specific context. You should not be afraid to duplicate code between contexts for them to be the clearest possible.

## The tools you *need* to understand your *needs*

Here are few of the tools we should use to truly understand our needs when building software:

- User meetings;
- Domain experts interviews;
- Event Storming (more on that later);
- US Mapping (more on that online);
- Impact Mapping: The goal of this workshop is to identify a goal, define actors, the impacts of each of the actors and the deliverables linked to these impacts.

The point here is to *speak to the actors*, and not to base our choices on assumptions and transcriptions.

Those tools and other communication medium can help us create an *Ubiquitous Language*.

> *Ubiquitous Language*: Common language spoken by every member of the team. Here, **two identical words should have the same domain meaning** and **two different ones should not designate the same concept**.

The *Ubiquitous Language* helps us align the source code with the domain. A domain expert should be able to understand the source code (the domain one, obviously).

## Focus on Event Storming

This type of workshop was not discussed on the [**Blue Book**](https://www.oreilly.com/library/view/domain-driven-design-tackling/0321125215/), but it is still a quite relevant tool.

You could use it as a way to grasp the big picture of your project, or to try to find insight about a specific part of your domain. It is a medium of communication for every team members, where we start from the objectives and take a step back.

There are many ways to conduct the workshop, but here is how we did it at the training. The example is for a betting application, with the team trying to do a Big Picture Event Storming.

The *Product Owner* tells the story of the product, based on his vision of it.

Every time someone find a domain event related to the story told, he writes it down on a sticky-note of any color (usually an orange one) and put it on a timeline.

> Example: Goal scored

Once the *Product Owner* finished his story, the team tries to associate a command (blue sticky-note) and an actor (yellow sticky-note) on every domain event. This process reverse the narration.

> Example: Score a goal, by a player.

The next step is to regroup the events/commands/actors in aggregates, the timeline can be ignored for the end of the workshop.

> Example: A "match" aggregate arise with the "Goal scored" event and the associated command and actor, plus other events related to the players.

The final step is to regroup aggregates in contexts.

> Note: We did use this tool in my current mission, but with a more anarchist twist: we started by adding every domain events that crossed our mind, then we filtered them. I personally find this introduction to be less productive since you spend most of the workshop sorting sticky-notes.

When doing this workshop, you should be careful to always use the *Ubiquitous Language*, and you can mark the uncertainties with red sticky-notes.

This is a really diminishing description, I urge you to read [*Alberto Bradolini*'s **Event Storming** materials](https://www.eventstorming.com/) for more information.

## Organize your inter-contexts communications

When using *DDD* at the enterprise scale, you need to set up inter-teams and inter-contexts strategies.

This is where the *Strategic Patterns* play they part.

> *Strategic Patterns*: Communication patterns used to organize the exchanges between contexts.

> *Upstream*: The upstream can be seen as the owner of the data, of the features.

> *Downstream*: The downstream can be seen as the client, he will usually be influenced by the upstream.

Here is a quick description of each of the patterns introduced during the training:

- *Big Ball of Mud*: Basically any situation where the communications go in every direction. Spoiler alert: it is bad;
- *Partnership*: When two contexts work together in a situation when if one fails, the other goes down with him;
- *Conformist*: One context comply to the other one's vision. The downstream does not have his word to say in the choices made by the upstream;
- *Customer/Supplier*: Like the *Conformist* approach, the downstream consumes the upstream's features but the downstream can makes requests for changes or new features;
- *Separate ways*: In the not-so-rare situation when you have two different solutions to the same problem, two contexts can decide to have zero communication and find their own way;
- *Published language*: Everyone follows a contract, a standard;
- *Open Host Service*: The upstream's provide an *API* that is consumed by everyone who needs it;
- *Shared Kernel*: This is the one that I personally dislike the most. It is basically sharing some code between multiple contexts, via a library for instance. I dislike it because I found it odd to have to share some code, I cannot think of any case where this is not a smell of a bad responsibilities split. Feel free to prove me wrong!

Now, if you want to protect your context from the outer world's noises, you could put an *Anticorruption Layer* (or *ACL*) in your inputs and outputs. You can do so just by making the communications from and to your *domain* going through an *interface*. Your infrastructure layer will have the responsibility to implements this interface and by doing so translate the messages for the world to understand.

To visualize the whole *Strategic Patterns* and contexts in play, you can draw a *Context Map*. This is a map of all of the contexts, their interactions and the type of those. Take [**Mathias Verraes**'s example](http://verraes.net/2014/01/bandwidth-and-context-mapping/) for a more in-depth look.

We have seen the patterns used to communicate between the contexts, now we will see the patterns used in the actual code.

## Organize your code

> *Tactical patterns*: Engineering patterns used to organize the code-base in order to reflect the domain.

Here is a quick description of the building blocks described in *Eric Evans*' [**Blue Book**](https://www.oreilly.com/library/view/domain-driven-design-tackling/0321125215/), please go back to this book if you want more information.

- *Value Object*: An object that has no identity, no lifetime and who is compared based on his values;
  - Example: the red color can be represented as an object with the *hexadecimal* value set to *"#FF0000"*;
- *Aggregate*: Special kind of *Entity*, has an identity and a lifetime. Can be composed of *Value Objects* and *Entities*, but not of other *aggregates*;
  - Responsible for the domain logic for a single *aggregate*, every action on the aggregate or the elements inside it must go through him;
  - Cannot directly communicate with other *aggregates*, those communications must go through a *service*;
  - Example: A person can be represented as an object with attributes and identify with his social security number. Actions on this person's tooth must go through the *aggregate* representing him, not directly on the tooth;
- *Entity*: An object with an identity unique inside the aggregate;
  - Example: The front-left wheel in a car;
- *Service*: Has no lifetime nor data. Used to synchronize aggregates;
- *Domain Event*: Used to communicate with the world. Can be seen as a *Value Object* representing a decision made.

Additionally, two patterns are usually used to manipulate *aggregates*:

- *Repository*: Used to manipulate collections of *aggregates*, to persist them for example;
- *Factory*: Used to build non-trivial *aggregates*. To be used when the *aggregate*'s constructor is not enough and the creation logic is too complicated.

## DDD and Software Architecture

**Loose coupling / high cohesion** is something you want to achieve when building an application with the *DDD* tools.

I will list some architectural patterns that you can use to do so, but please remember to be **practical**. You can combine all of those patterns, but it will create complexity. You should always ask yourself if it is worth the investment before implementing one of those.

### Hexagonal architecture / Ports and Adapters

> *Hexagonal architecture*: Also known as *Ports and Adapters*. Pattern coined by [Alistair Cockburn](https://twitter.com/totheralistair) aiming to isolate the domain concerns from the infrastructure ones.

![Hexagonal architecture / Ports and Adapters](https://github.com/ThomasFerro/readmes/blob/master/posts/10-summary-of-a-four-days-ddd-training/hexagonal-architecture.png)

> Credits to *Maxime COLIN* for the schema on the *Elao*'s blog.

This pattern is quite simple to put in place. It is mainly about interfacing the interaction with the outer world.

You have two kind of ports: 

- "In": the entry points to the domain;
- "Out": the exit points.

For instance, you will have an entry port as an interface with the actions available in your domain. This interface could be implemented by a *REST API*, a *Message Queue* or event a *Command Line Interface*, the domain concern will not change.

The same goes for the exit points, no matter where you are retrieving or persisting your data, the domain is not impacted.

A more advanced architectural pattern based on the same principles is *Robert C. Martin*'s [**Clean Architecture**](https://www.goodreads.com/book/show/18043011-clean-architecture). I wont go into much details here since we did not see it in the training, but I do think that it is a great way to respect the *Clean Code* principles.

![Uncle Bob's Clean Architecture](https://github.com/ThomasFerro/readmes/blob/master/posts/10-summary-of-a-four-days-ddd-training/clean-architecture.jpeg)

### Event Sourcing

> *Event Sourcing*: Architectural pattern focusing on the path that led us there instead of where we are right now.

![Event Sourcing schema](https://github.com/ThomasFerro/readmes/blob/master/posts/10-summary-of-a-four-days-ddd-training/event-sourcing.png)

It all start with an *aggregate* receiving a *command*. Based on this *command*, the *aggregate* will make decisions in the - wait for it - *decision methods*.

Those *decision methods* will then return events.

Those events will be applied to the *aggregate* in order to modify his current state. This process is done in *application methods*.

The events are not only applied, they can also be sent outside the *aggregate*, stored and published for side-effects. For instance, an event can trigger a mail notification when published.

You might need a concrete example, here is one in the context of our Betting app: 

- A goal is scored, represented as a *command* in the **Match** *aggregate*;
- The *decision method* handling the command, based on the data inside the command, decide to send the following event: **ScoreGoaled** with the player who scored, the time, etc;
- This event is applied to the current match, updating his state;
- It is also stored and published, triggering the notification system for the users who bet on the match.

This example does not go into details about how to loop in the *decide/events/apply* iteration, but know that there are use cases when you will need to.

We talked about storing events, but what about persistency ? Do we still be using a relational database ?

> *Event Store*: Persistency mechanism where you are not storing the current state of a data, but all the events that led to this state.

Take for example an *aggregate* representing a car. You can choose to store the current state of the car, of his wheels, engine, etc in a relational database, or you can store **every events that occurred on the car**. May they be a *gas refuel*, *oil change*, *wheel change*, etc.

You can then rebuild the *aggregate* by replaying all of the events since they are idempotents.

This approach works especially well with *DDD* since you can use the *domain events* already discovered in workshop such as Event Storming. Events applied on *aggregates* should match 1:1 those *domain events*.

You can go even further by combining *Event Sourcing* with another mechanism, *CQRS* who come in handy when dealing with performance issues.

### CQRS

> *Command Query Responsibility Segregation* (or *CQRS*): Split your application in two components: the *Command* to update the data and the *Query* to retrieve it.

![CQRS schema](https://github.com/ThomasFerro/readmes/blob/master/posts/10-summary-of-a-four-days-ddd-training/cqrs.png)

This pattern is especially useful when the update requests and the read ones happen at a very different rate.

For instance, take a booking application. You can have a huge traffic, meaning a lot of query request, while only a small portion of those visitors end up booking an actual trip. In this situation, having a lightweight application managing the *commands* can be enough, and you can focus your efforts on the *query* application that will need to manage the load.

To oversimplify the process, you can put it this way :

1. A *command* is played on an *aggregate*;
2. This command generate *events*, modifying the *aggregate*;
3. Those events can be stored, but they especially are catched by the projections impacted;
4. Those projections update themselves, updating the *queries*.

The clients only consume projections, which is faster than having to rebuild the *aggregates* every time.

## Back to the training

The training did a great job introducing the tools and concepts while making us practice with a lot of examples.

I tried to put in this article every major themes, consider it a quick go-to when trying to explain what is *DDD*.

Events and projections persistency, transactions, events lifetime and more details about the implementation of a *CQRS / ES* (*Command Query Responsibility Segregation* / *Event Sourcing*) architecture are not discussed here. Those advanced concepts are yours to discover and practice!

I want to thank [**Symbol-it**](https://symbol-it.fr/), the company I work for, for the logistic and for listening to my personal training wishes.

And of course, a huge thank you to *Florent Pellet* for sharing his knowledge and spending those enlightening four days with us!
