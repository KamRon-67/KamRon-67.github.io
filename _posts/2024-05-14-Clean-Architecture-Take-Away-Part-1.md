---
title: "Clean Architecture Take Away Part 1"
description: "Taking the time to process what I have read from books and the community"
tags: [Clean Architecture, C#, Code]
categories: [Blogging]
---

## The Take away

Over the last few months, I have reviewed clean architecture, the book, and various codebases. Then, I engaged with the tech community around me on Twitter. Currently, my takeaway is that clean architecture is just an implementation of software principles, allowing for the adoption of various architectural styles such as layered, service-based, and event-driven.

Many people I discussed with, who were not fans of clean architecture, still pointed to many of the same principles that clean architecture promotes. To do this, I had to play as a conscientious objector in the Uncle Bob War. I am here for the software views only. That aspect colored many people's views on the phrase "clean architecture." My response most of the time was “Never meet your heroes” or “Chew the meat and spit out the bones.”

Many people who have videos talking about clean architecture would implement CQRS. When I first started learning about clean architecture, I mistakenly thought that was it. Now I think CQRS is just a popular way to demonstrate the design principles of clean architecture, namely SRP (The Single Responsibility Principle), OCP (The Open Closed Principle), LSP (The Liskov Substitution Principle), ISP (The Interface Segregation Principle), and the most important principle, DIP (The Dependency Inversion Principle).

The book "Dependency Injection: Principles, Practices, and Patterns" states DIP as follows: “The Principle states that higher-level modules in our applications shouldn’t depend on lower-level modules; instead, modules of both levels should depend on abstractions.” Also, “The relationship between the dependency inversion principle and dependency injection is that DIP prescribes what we would like to accomplish. Not only does the Principle prescribe loose coupling, it states that abstractions should be owned by the module using the abstraction. In this context, owned means that the consuming module has control over the shape of the abstraction, and it is distributed with that module, rather than with the module that implements it.”

Bringing it back to my thoughts about clean architecture: In the core section, if you have an email interface that is used in handlers, Core owns the abstraction and is used all over; it lives in the core. The implementation happens in the infrastructure section and is attached to a class that implements it. The handlers in the core never talk to the class implementing the email functionality.

That makes sense but how would this same thing work in Python?
