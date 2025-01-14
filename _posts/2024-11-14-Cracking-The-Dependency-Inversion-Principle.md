---
title: "Cracking the Dependency Inversion Principle (DIP)"
description: "In this post, we’ll dive deep into the Dependency Inversion Principle (DIP), a key SOLID principle that enhances your code’s flexibility, maintainability, and testability"
tags: [DIP, Kata, C#, Code]
categories: [Blogging]
---

### Cracking the Dependency Inversion Principle (DIP)

Dependency Inversion Principle (DIP) might sound intimidating, but it’s a powerful tool to improve the flexibility, testability, and maintainability of your code. Here’s the main idea: **high-level modules** (your core business logic) shouldn’t depend on **low-level modules** (like specific services or infrastructure details). Instead, both should depend on **abstractions** (interfaces or abstract classes).

In other words, don’t make your code rely directly on concrete implementations. This allows your modules to be easily swapped or updated without touching everything else in the codebase.

---

### The Basics of DIP: Keep It Abstract

- **Program against abstractions** rather than specific implementations.
- **High-level modules define the abstraction** based on their needs; don’t let a specific implementation force unnecessary methods into your interfaces.
- **Keep interfaces focused** on essentials; avoid "interface pollution" where too many methods bloat an interface just for one implementation’s needs.

---

### Why Bother? DIP’s Payoff in Real Projects

DIP brings serious benefits:

- **Maintainability**: Modules built on abstractions are easier to refactor and update. Changing one component doesn’t require a codebase-wide overhaul.
- **Testability**: Abstractions let you swap in mocks and stubs, making it easier to unit test without complex setup.
- **Scalability**: DIP makes it easy to add or update modules without major rework.

---

### DIP in Action: The Battery Analogy

Imagine a toy car that only works with one type of battery. If that battery’s not available, you’re out of luck. But if the car just needs “a power source,” it can use any compatible battery, making it much more flexible.

DIP does the same thing for your code. By programming against an abstraction (e.g., an interface for “logging”), your code becomes way more adaptable, just like that battery-swappable toy car.

---

### Breaking DIP: A “What Not to Do” Example

Here’s a classic example of DIP gone wrong in C#. Here, `OrderProcessor` is directly dependent on `FileLogger`, a concrete implementation. This violates DIP:

csharp

Copy code

```csharp
public class FileLogger
{
    public void Log(string message)
    {
        Console.WriteLine("Logging to a file: " + message);
    }
}

public class OrderProcessor
{
    private FileLogger _logger;

    public OrderProcessor()
    {
        _logger = new FileLogger();
    }

    public void ProcessOrder()
    {
        // Processing order
        _logger.Log("Order has been processed.");
    }
}
```

**What’s wrong here?**

- `OrderProcessor` (the high-level module) is tightly coupled to `FileLogger` (the low-level module).
- Changing `FileLogger` to something else (like a `DatabaseLogger`) would require modifying `OrderProcessor`, which isn’t flexible or scalable.

---

### Fixing It: Refactor to Follow DIP

By introducing an interface, `ILogger`, we can let `OrderProcessor` depend on an abstraction rather than a specific implementation. Here’s how it looks:

```csharp
public interface ILogger
{
    void Log(string message);
}

public class FileLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine("Logging to a file: " + message);
    }
}

public class OrderProcessor
{
    private readonly ILogger _logger;

    public OrderProcessor(ILogger logger)
    {
        _logger = logger;
    }

    public void ProcessOrder()
    {
        // Processing order
        _logger.Log("Order has been processed.");
    }
}
```

**Why this works better**:

- Now `OrderProcessor` is only dependent on `ILogger`, which makes it super easy to swap in other loggers (e.g., `DatabaseLogger`) without any changes in `OrderProcessor`. That’s flexibility in action.

---

### DIP and Dependency Injection: A Dynamic Duo

DIP sets up what we want to achieve (abstraction-based dependencies), and **Dependency Injection (DI)** helps us get there. DI allows us to “inject” dependencies from the outside, so `OrderProcessor` can receive any `ILogger` implementation instead of creating one itself.

### Avoiding Common DIP Pitfalls

1. **Interface Pollution**: Don’t add unnecessary methods to an interface just to make one specific implementation happy. Keep interfaces focused and lean.
2. **Overusing Abstractions**: Not everything needs an abstraction. Creating interfaces for trivial things can add unnecessary complexity.
3. **Ownership of Abstractions**: Ideally, the module that needs the abstraction (e.g., `OrderProcessor`) should define it, rather than letting the low-level module control the abstraction.

---

### Get Hands-On: Practicing DIP with a Kata

To apply what you’ve learned, try this [kata](https://gist.github.com/jhubermilliman/3e304cf712bed4e97f5b7d60d28fe230). Start with a timer to give yourself a bit of pressure and try a test-driven development approach: write a failing test, add code to pass it, then refactor if needed.

### DI Container Kata: A Hands-On Exercise with DIP

In this kata, you’ll build out a basic DI container and add features step by step to strengthen your understanding of DIP.

#### Requirements:

1. **Register and Resolve Types**: Each `Resolve` call should return a new instance.
2. **Singleton Registration**: Make sure singletons return the same instance on each `Resolve` call.
3. **Constructor Injection**: Add dependency injection through constructors.
4. **Handle Circular Dependencies**: Ensure circular dependencies throw an exception rather than causing an infinite loop.

### Sample Implementation

Here’s some minimal inspiration for your DI container setup in C#:

```csharp
public class MyContainer
{
    private Dictionary<Type, Type> _registrations = new();

    public void Register<TInterface, TImplementation>() where TImplementation : TInterface
    {
        _registrations[typeof(TInterface)] = typeof(TImplementation);
    }

    public TInterface Resolve<TInterface>()
    {
        var implementationType = _registrations[typeof(TInterface)];
        return (TInterface)Activator.CreateInstance(implementationType);
    }
}

```

### Tests for the DI Container

These tests check if the container meets the requirements above, like handling singletons and resolving different instances. Adding tests for edge cases, like thread safety and missing registrations, can make it even more reliable.

```csharp
public class DIContainerTests
{
    [Fact]
    public void when_registered_should_resolve_new_instance()
    {
        var ioc = new MyContainer();
        ioc.Register<IFoo, Foo>();
        var instance1 = ioc.Resolve<IFoo>();
        var instance2 = ioc.Resolve<IFoo>();

        Assert.NotSame(instance1, instance2);  // Different instances
    }

    [Fact]
    public void when_registered_as_singleton_should_resolve_same_instance()
    {
        var ioc = new MyContainer();
        ioc.RegisterSingleton<IFoo, Foo>();
        var instance1 = ioc.Resolve<IFoo>();
        var instance2 = ioc.Resolve<IFoo>();

        Assert.Same(instance1, instance2);  // Same instance
    }
}
...

[Fact]  
public void when_registered_a_type_resolve_should_return_a_new_instance_of_that_type()

...

[Fact]  
public void when_registered_a_type_as_singleton_type_should_return_the_same_instance_of_that_type()  

...

[Fact]  
public void when_registered_twice_as_a_singleton_ioc_should_throw_exception()  
```

---

### Wrapping Up

Learning DIP can make your code more adaptable and resilient to changes. Practicing these principles with katas and exercises will build your skills and help you write cleaner, more scalable code. Happy coding, and keep experimenting!
