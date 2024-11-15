---
title: "Intro To DI, Aggregation Association And Composition"
mermaid: true
tags: [C#, Aggregation, Association, Composition, Inheritance, Dependency Injection]
categories: [Blogging, Dependency Injection]

---

## Why are we here
Sometimes while learning software concepts. I like to dig a few layers deeper to understand better. One of my older mentors used to say, “You have to bring books to the books sometimes.” While reading Dependency Injection principles, practices, and patterns, another book kept popping up into my head. The object-oriented thought process. The dependency injection book has been a great read. I even recommend it. Yet as my reading progressed. My early definition of dependency injection, "A set of software design principles and patterns that enables you to develop loosely coupled code." I asked myself. What is the smaller subset of this principle? My experience in software has been. That if it is complex. You can probably break it down into smaller components.

I will stop there and focus on how composition and inheritance kept popping up. I found many views on the subject. Even the object-oriented thought process book tells the reader, "the views it had could be contested." So using that and other sources, we learn that dependency injection relies on composition. It is a way to implement Inversion Of Control. IOC it and dependency injection are related. Dependency injection is one of many ways to implement IOC. So I will stop there and focus on how composition aids us in DI.

So what is composition? Why should we use it? I got many definitions depending on where I looked. Sticking with the book The object-Oriented Thought Process, they define composition as an object that is built from other objects.

## Why use it?
First, why should we use it? We do not have many options if we want to build classes from other classes. There is inheritance and composition. Many people have issues with inheritance and are very vocal. I am not, I think it's a tool. If it can help you, use it. If it doesn't fit your needs don’t use it. Can’t speak for everyone but my anecdotal experience from college was you learned only inheritance. So armed with that knowledge you would force that into everything you did through inheritance. Even if it was not needed. This brought up another underlying concept of has-a vs is-a.

The same way in physics you have molecules, atoms, protons, neutrons, electrons, and the army of quantum particles like quarks. Is the same way many software concepts can come together and form bigger concepts. Back to Is-a and Has-a. The Is-a relationship is represented by inheritance, this is a separate concept. Inheritance is tightly coupled to polymorphism. Let's remember that inheritance allows a class to inherit the attributes and methods of another class. This allows the creation of brand new classes by abstracting out common attributes and behaviors. This is why when a class inherits from a parent class it is of the same type.


## Code example of inheritance
```csharp
public class Mammal
{
    public void Sleep()
    {
        Console.WriteLine("zzzz");
    }

    public void WarmBlood()
    {
        Console.WriteLine("I have warm blood");
    }

    public virtual void MakeNoise()
    {
        Console.WriteLine("kaa kaa");
    }
}

class Dog : Mammal
{
    public override void MakeNoise()
    {
        Console.WriteLine("Bark Bark");
    }
}
```

```mermaid
classDiagram
      Mammal <|-- Dog : Inheritance
      Mammal: +Sleep()
      Mammal: +WarmBlood()
      Mammal: +MakeNoise()
      class Dog{
          +MakeNoise()
      }
 ```

In the above example, the dog class has two types, of course, dog but mammal as well. The dog class has access to the sleep method and anything in the mammal class that is not private. That itself is the problem. The dog class is tightly coupled to the mammal class. Any changes there now affect the dog class. If this is planned. That is not a problem. Sometimes this can lead to changes being hard or unexpected behavior when the inheritance tree starts to grow.

Lots of people will just say never use inheritance or favor composition over inheritance. I think this article covers why that is bad advice. Inheritance and composition are important techniques. When it comes to building OO systems. Take a moment to understand the strengths and weaknesses of both and to use each at the right times. This link here is a good example of

## Example of the concept "favor composition over inheritance"?
[link](https://softwareengineering.stackexchange.com/questions/65179/where-does-this-concept-of-favor-composition-over-inheritance-come-from/65209#65209)

So ending our talk on inheritance, remember this is not a damnation of that, just a deep dive. Back to composition.

## Composition
With composition, a class has a field of another class interface or class. The relationship is has-a. With composition, you design your objects around what they do. Using the mammal dog example from above it would be structured like this.

```csharp
class Dog
{
 Mammal mammal;
 ...
}
```

Here we would use the mammal object to use the make MakeNoise method. The problem here is we can not override that method. So to use composition would not be the best idea but you see how it works. While a bad example causes the example above was showing you how you can override a method. This is in the spirit of what we are shooting for but we can do better.

```csharp
class A
{

}

class B
{
    var id = new A();

    id.method();
}
```

Composition is a foundation of dependency injection. Let's drill into this more before we move on to D.I. In short, whenever a particular object is composed of other objects. Those objects are included as object fields. The new object is known as a compound, or aggregate, or composite object. There are a few flavors of composition having a has-a relationship. The two types of composition are aggregation and associations. Note that Composition is an area where the question of which came first. Some think that composition is a form of association, and others think association is a form of composition.

## Aggregation Association, and Composition
Aggregation and composition are types of association. Based on the object-oriented thought process, "In this book, I Consider aggregation and association to be types of composition, all though there are varied opinions on this" looking at StackOverflow this is very true. This is even boiled down to just association and aggregation. The main difference being with aggregation you normally see only the whole and in associations, you normally see parts that make up the whole.

A "owns" B = Composition: B has no meaning or purpose in the system without A

A "uses" B = Aggregation : B exists independently (conceptually) from A

A Text Editor owns a Buffer (composition). A Text Editor uses a File (aggregation). When the Text Editor is closed, the Buffer is destroyed but the File itself is not destroyed.

## Composition over Inheritance
Covered above many blogs and posts. Many others want you to favor composition over inheritance, due to its "is-a" VS "has-a relationship". The main issue being inheritance is overused in many situations and could lead to tight coupling. When changes in the parent class break functionality in the child classes. Then we have a design issue. What should the structure of the hierarchy look like and when new components are needed how do you integrate them into your current system.

The problem with this example is not in its ability to explain contracts but in how they could be extended. If we needed a reptile with the same properties as a mammal. We would only have a few options. Make lizard a subclass of mammal. This is bad practice lizards are not warm-blooded. We would have methods that do not fully apply to us. I have personally seen this in a few code bases. Create a reptile superclass with the same properties as a mammal, this is also bad in my eyes as now we have code duplication. Moving from an is-a to a has-a relationship we move away from this complexity, the fragile base class problem.

You can learn and use DI without knowing this in great depth. I just find it helps to know the tools you are using.