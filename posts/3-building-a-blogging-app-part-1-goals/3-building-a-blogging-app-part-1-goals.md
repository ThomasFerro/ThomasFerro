# Building a blogging application part 1 - The goals

## There was an idea..

> ...  to bring together a group of remarkable technologies, see if they could become something more.

Behind this not very subtle reference lies the whole idea of this series of posts.

### Genesis

I think that a little of backstory is necessary to fully understand the shift of mind that leads to the making of this post.

We are, with a friend of mine, working at an IT services company and are currently in a mission within a little team of software engineers for a big retail company.

Even though we are passionate about our job and the Web Development in general, there is something in this specific mission that feels wrong for us.

We have been trying to find the main reason behind this negativity, and we may very well have found it : this is project driven by the technical choices.

It started out last year with a POC (spoiler alert, all POCs goes in production eventually), and since then the team only speaks about *microservices*, *Java*, *Spring*, *SpringBoot*, *Node.js*, *Puppeteer*, etc...

The results of it all being that we desperately try to make the *domain concerns* fit this technological knot bag.

### The search for a better world

After speaking about this issue with our technical director and searching for resources on the Internet, we came across the *Domain Driven Design* principles and references.

I also have started to read more and more Clean Code and Architecture literature (mostly *Robert C. Martin*'s and *Kent Beck*'s books).

Though I have not finish reading the Blue Book yes (*Domain-Driven Design: Tackling Complexity in the Heart of Software* by *Eric Evans*), I am convinced that this is the right path to follow in order to solve our issues.

I already presented the concepts to the team, but I wanted to have a first experience with it before applying the principles in our application.

And so, here I am, **building a blogging application** in order to learn and try to apply these practices !

## Axes

The main three axes for the project are the following :

1. The domain
2. DevOps
3. Web Dev

### The domain

As explained before, my main frustration as a Web Developer is that we do not think about the domain as the most important concept.

We are not here to play with the technologies for the fun of it, but to create domain value.

My main focus during this project will be to apply the principles that I learn about DDD.

### DevOps

I will admit it, I never have given myself the time to do more that scratching the surface about all of the DevOps area.

In this project, I really want to leave my comfort zone and take full advantage of this relatively new domain.

May it be the setup of a complete CI/CD pipeline, or the more advanced Docker possibilities.

I genuinely think that this is the most difficult part for a self learner with no previous experience in that field to get right. 

There is a lot (and I mean **a lot**) of resources out there. But to be able to filter out the deprecated stuff and to find the most suitable solution is rather hard.

### Web Dev

This axis is more "for the fun" than for the real value of it.

There are some subtopics about Web Development that I really want to dig into, such as **the Server Side Rendering** in Vue.js (pros and cons, how to set it up, ...).

I also am interested in the making of a Web Application using only native Web Components.

This is the part that is likely to come last and least developed, as it is not the main point of the whole project.

## What is coming next

I do not want to explain every details right now. Just know that I will post updates about the project as it goes.

I plain to log every interesting steps, may they be failures (via post-mortems) or successes.

It will be an honor and a pleasure to be able to start conversations and debates from those posts, so feel free to comment and share your opinion !

Thanks for reading, see you soon 😁
