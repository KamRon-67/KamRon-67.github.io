---
title: "Simplify Your Clean Architecture Practice: A Framework Without MediatR Overhead"
description: "When working with an architectrual kata lets see if we need MediatR"
tags: [Clean Architecture, MediatR, Architecture Kata, Minimal APIs]
categories: [Blogging]
image: assets/img/MediatRComic.png
---

### Simplify Your Clean Architecture Practice
	

I stopped using MediatR just to practice clean architecture fundamentals. If you are working a simple project or a quick kata, you need things to be available. You need focus, not complex abstractions. Discover an explicit framework designed to get your hands dirty. 
Many clean architecture examples, often drawing inspiration from Uncle Bob’s famous diagrams showing the flow of control, lean on the popular MediatR library. 

```csharp
app.MapPost("/api/posts", async (IMediator mediator, Post post) => 
{
    var createPost = new CreatePost { PostContent = post.Comments };
    var cratedPost = await mediator.Send(createPost);
    return Results.CreatedAtRoute("GetPostById", new { createPost.Id }, createPost);
});
```

It’s a solid library for implementing the Mediator pattern, effectively decoupling how objects interact by routing communication. Through a central mediator object rather than direct calls. It helps keep handlers isolated, but this can also be achieved cleanly using direct dependency injection. Something like this. 

```csharp
app.MapPost("/api/posts", async (Post post, ICreatePostHandler createPostHandler) =>
{
    var createPost = new CreatePost { PostContent = post.Content, PostComments = post.Comments };
    var createdPost = await createPostHandler.Handle(createPost);  
    return Results.Created($"/api/posts/{createdPost.Id}", createdPost);
});
```

### The Case Against MediatR

Let's be honest, for focused practice, the added ceremony of setting up request, handler, and MediatR objects can be counterproductive. Taking MediatR out of the framework reminded me of many files I had to touch setting this up.  For real world projects there is a case for MediatR. Especially if you intend to use its pipeline feature to implement shared behaviors. The creator of MediatR mentioned when it is time to use it and when to leave it alone. So this sparked an idea, how would I create a clean architecture set up specifically streamlined for katas. This led to the decision to build a setup without MediatR.

I have gotten use to including MediatR in this kind of set up so much so I hadn’t seriously considered alternatives or rolling out my own solution. Doing a quick search I did not see any implementation using minimal api’s straying away from this pattern. Using minimal api’s we have straightforward dependency injection, so why couldn’t API endpoints just directly call registered application service interfaces. 
Exploring the Alternative: Direct Dependency Injection
I decided to build it. The first step was ripping MediatR out of one of my existing lean clean Architecture projects. Predictably, this unleashed a wave of compile errors. Luckily, a solid suite of tests was already in place. These tests became the driving force behind the refactor. Moving in this direction, the set up is arguably over engineered intentionally, to explore different patterns. Removing the library proved to be a fantastic exercise in simplifying the interaction logic.

The process involved systematically working through test failures. Primarily integration tests sending HTTP requests to API endpoints /api/posts and unit tests focused on handlers until everything passed again. This test-first refactoring ensured the core functionality remained intact while the communication mechanism changed. These integration tests now serve as proof that the endpoints, application logic, and database interaction work correctly without the mediator. 

The result? A Clean Architecture framework tailor-made for katas. It retains the essential layered structure but replaces MediatR's indirection with explicit, direct calls via dependency injection. This makes the flow of control instantly traceable and slash setup time, letting you focus purely on practicing your domain modeling and application logic.

Go ahead and try an architectural kata!
    • [Explore the framework.](https://github.com/KamRon-67/clean-architecture-for-katas)
    • See the explicit flow: Less magic, more clarity.
    • Get started faster: Less boilerplate for your practice sessions.

Contributions or feedback are always welcome!



