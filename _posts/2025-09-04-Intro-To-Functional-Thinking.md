---
title: "Changing your thinking from Imperative to FP "
description: "To change your thinking from Imperative to FP"
tags: [Functional Programming]
categories: [FP]
---

This is not an intro to functional thinking; this is more of a good to know while diving into FP. There is a joke: at the high level, it's just a way to replace design patterns and many tricks we use in OOP with functions, while avoiding state mutation. (This is a joke, but it gets to the heart of the matter: it's about shifting your mindset and embracing functions.) As you read the article, you will see there is more to it.

The TLDL:
*   **Functions as first-class citizens** - Functions take other functions as input or return a function as output.
*   **Delegates** - are a type-safe function pointer.

### Functions as first-class citizens and HOF's

Outside of LINQ calls, I don't run across functional programming in the wild often in C# applications. There is a conversation that masked itself as FP vs. OOP. Well, it really was more of an imperative vs. FP and not OOP vs. FP, in my anecdotal experience.

There are a few things that will help you a lot when you are getting your FP chops. Functions are first-class citizens. This did not really mean much to me at first. Functions as first-class citizenship allows you to use functions in more versatile ways, like using them as inputs or outputs of other functions, assigning them to variables, or storing them in collections.

You can run this quick example in the REPL tool.

A super small quick example.

Look into REPL tool; it is not something I am planning to cover here, but it is great for following along and keeping things simple for now. Run this line by line.

```csharp
Func<int, int> doubleFunc = x => x * 2; 
var range = Enumerable.Range(1,3); 
var doubles = range.Select(doubleFunc);
// In a REPL, running 'doubles' alone would display the contents (e.g., [2, 4, 6])
```

#### What is going on here?

We first declare a function. The value `x` is the input. We then make a small list (`range`) then invoke the `Select` extension. The LINQ `Select` operator is a projection. So, in short, you get a new object. We are able to use the data in `range` and **not mutate the original collection**. Now we just use the `doubleFunc` function as an argument.

The `Select` method is an extension on `IEnumerable`; we gave it the `range` and `doubleFunc` as arguments. This was a small example of functions being first-class citizens in C#.

### What is a delegate?

Delegates are type-safe function pointers. Type-safe here means that a delegate is strongly typed: the types of the input and output values of the function are known at compile time, and consistency is enforced by the compiler.

Creating a delegate is a two-step process: you first declare the delegate type and then provide an implementation. (This is analogous to writing an interface and then instantiating a class implementing that interface.)

The first step is done by using the `delegate` keyword and providing the signature for the delegate. Once you have a delegate type, you can instantiate it by providing an implementation.

Again, a delegate is just an object (in the technical sense) that represents an operation. Just like any other object, you can use a delegate as an argument for another method. **Delegates are the language feature that makes functions first-class values in C#.**

We are going to have a more verbose breakdown. So, here is a bigger code example. Let's break it down.

#### Code Block Example

```csharp
using System;

// 1. Declare the delegate type (the "blueprint")
public delegate int MathOperation(int a, int b);

public class Program
{
    // 2. Create methods that match the delegate signature
    public static int Add(int x, int y)
    {
        return x + y;
    }
    
    public static int Multiply(int x, int y)
    {
        return x * y;
    }
    
    // 3. Method that takes a delegate as parameter
    public static void CalculateAndPrint(int a, int b, MathOperation operation)
    {
        int result = operation(a, b);
        Console.WriteLine($"Result: {result}");
    }
    
    public static void Main()
    {
        // 4. Instantiate delegates
        MathOperation addDelegate = Add;
        MathOperation multiplyDelegate = Multiply;
        
        // 5. Use the delegates directly
        Console.WriteLine("Using delegates directly:");
        Console.WriteLine($"5 + 3 = {addDelegate(5, 3)}");
        Console.WriteLine($"5 * 3 = {multiplyDelegate(5, 3)}");
        
        // 6. Pass delegates as parameters
        Console.WriteLine("\nPassing delegates to methods:");
        CalculateAndPrint(10, 4, addDelegate);
        CalculateAndPrint(10, 4, multiplyDelegate);
        
        // 7. You can also use method group conversion (shorter syntax)
        Console.WriteLine("\nUsing method group conversion:");
        CalculateAndPrint(8, 2, Add);
        CalculateAndPrint(8, 2, Multiply);
    }
}
```

This is a lot of code, so let's look at the core of it:

```csharp
public delegate int MathOperation(int a, int b);
```
This single line is our contract. It declares that any method that matches this pattern—taking two integers and returning an integer—can be assigned to a `MathOperation` delegate. The C# type system handles the enforcement of this contract.

Look at the `CalculateAndPrint` method:

```csharp
    public static void CalculateAndPrint(int a, int b, MathOperation operation)
    {
        int result = operation(a, b);
        Console.WriteLine($"Result: {result}");
    }
```
This is the magical part. `CalculateAndPrint` doesn't know (or care) if `operation` performs addition, multiplication, subtraction, or anything else. It only knows it will take two `int`s and give back one `int`.

*   You can notice we are using **Polymorphism** as the `operation(a, b)` can change as long as it adheres to the signature type.
*   We also have **Encapsulation** as `operation` is the *what to do* but not the *how to do it*. We would have to pass in a new method to change the behavior.
*   We are also getting some **Abstraction** as the delegate is a contract, and the specific methods (`Add`, `Multiply`) are concrete implementations of that abstract contract.

As you noticed, I am still using terms you learned from OOP. Like I said earlier, there is not really an OOP vs. FP debate. They overlap a lot. You can use FP techniques in your OOP code.

The flexibility shines when you pass delegates as arguments:

```csharp
        MathOperation addDelegate = Add;
        MathOperation multiplyDelegate = Multiply;
...
        CalculateAndPrint(10, 4, addDelegate);
        CalculateAndPrint(10, 4, multiplyDelegate);
```

There have been tons of times I have had to duplicate code when I could have made a delegate and passed a function.

### So many different ways to write delegates

I showed you the older way just so you know what is going on under the hood. Most of the time, you will see **`Func`** and **`Action`** as a short hand. There will be times it makes sense to define your own delegates.

**`Action`** is a pre-defined generic delegate type that represents a method that **does not return a value (void)** and can take zero to 16 input parameters.
```csharp
Action myAction = () => Console.WriteLine("No parameters, no return.");
Action<string> printMessage = (msg) => Console.WriteLine(msg);
```

**`Func`** is a pre-defined generic delegate type that represents a method that **returns a value** and can take zero to 16 input parameters. The **last** type parameter in a `Func` declaration always specifies the return type.
```csharp
Func<int> getNumber = () => 42;
Func<int, int, int> addNumbers = (a, b) => a + b;
```

### Another code wall of code but using a predefined delegate

Here is the previous example rewritten, completely removing the need for the custom `MathOperation` delegate by using the built-in `Func<T, TResult>`.

```csharp
using System;
// Note: The custom delegate declaration is REMOVED.
// public delegate int MathOperation(int a, int b); // Removed

public class Program
{
    // 2. Create methods that match the signature (they remain the same)
    public static int Add(int x, int y)
    {
        return x + y;
    }
    
    public static int Multiply(int x, int y)
    {
        return x * y;
    }
    
    // 3. Method that takes a delegate as parameter
    // The type is changed from MathOperation to Func<int, int, int>
    public static void CalculateAndPrint(int a, int b, Func<int, int, int> operation)
    {
        int result = operation(a, b);
        Console.WriteLine($"Result: {result}");
    }
    
    public static void Main()
    {
        // 4. Instantiate delegates using Func<int, int, int>
        Func<int, int, int> addDelegate = Add;
        Func<int, int, int> multiplyDelegate = Multiply;
        
        // 5. Use the delegates directly (invocation syntax is the same)
        Console.WriteLine("Using delegates directly:");
        Console.WriteLine($"5 + 3 = {addDelegate(5, 3)}");
        Console.WriteLine($"5 * 3 = {multiplyDelegate(5, 3)}");
        
        // 6. Pass delegates as parameters
        Console.WriteLine("\nPassing delegates to methods:");
        CalculateAndPrint(10, 4, addDelegate);
        CalculateAndPrint(10, 4, multiplyDelegate);
        
        // 7. You can also use method group conversion (shorter syntax)
        Console.WriteLine("\nUsing method group conversion:");
        CalculateAndPrint(8, 2, Add);
        CalculateAndPrint(8, 2, Multiply);
        
        // BONUS: Using an anonymous method/lambda expression
        Console.WriteLine("\nUsing a Lambda expression:");
        CalculateAndPrint(15, 5, (x, y) => x / y);
    }
}
```

As you see, the delegate is just a placeholder for the logic of the function, basically. It allows you to pass behavior around as data.