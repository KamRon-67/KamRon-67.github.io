---
title: "You're Not a Code Writer Anymore. Now What?"
description: "AI is writing code for us now"
tags: [AI]
categories: [AI]
---

### Workflow thoughts

We now can have AI create features as needed, but the new problem is code bloat. 
Given no instructions your generated code will be unsecured and most likely a spaghetti mess. 
We mitigate that with architecture and tests. You can use an MCP server but we will keep it simple. 
If you have an existing architecture you can have AI review it, have it create a markdown file for itself for feature creation. 
At this point I would highly recommend integration testing, more and more the test will be for the humans.Code coverage at this point is meaningless to me, we want to now make sure our features are covered by tests and they break when the feature breaks. 

You are going to break existing features. It’s inevitable with the tools we are using, our new goal is to know asap. A lot of AI tools will look at the compiler errors and fix that in a loop but it will not verify existing workflows are not broken long as the code compiles. When we have failing tests we can feed that back into the model as well. Don’t forget you can’t trust the models it will sometimes change the test to pass and the test would lose its value.

A few years of hands on working with AI, I have learned it needs guard rails. At every level. I allowed it to think for me, that lasted a week tops. Most of the output, talked in circles, the long form content would have the “blogger sound”. In my personal projects, to get any real value out of AI. I have to give it a complete feature and I will have it mimic my style limiting the cognitive load and code bloat. It can create complex features that are hard to support, architecture is now a cognitive shield.

This sector is moving very fast, I remember everybody was working on a code generation tool. Now we have established code generation tools, if you can think of something you can have AI run in a loop to create whatever you want. We are not talking quality but it will do the flow you asked for. The context windows are getting bigger so it’s not crazy to have AI create an end to end feature. I hate to admit that AI has been more useful that expected. We just have to accept code bloat and more frequent PRs as a cost.

A lot has changed since I wrote part 1, at that time I wanted you to understand so you can type it out. You now need to understand since you will review code you did not write. You are losing part of the learning process and that was writing. Based on what I am seeing in a few corporate workflows you will most likely prompt a large portion of your work. So you will need to be explicit while prompting telling it the boundaries of this feature. I have seen from time to time the generated code will ignore what is in place if not given context. Examples being database calls can only happen in the infrastructure layer and can not happen in the controller. It is now on you to review and call things out and push back as needed.
