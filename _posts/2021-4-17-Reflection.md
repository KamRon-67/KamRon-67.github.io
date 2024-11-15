---
title: "An Example of Reflection"
description: "I understand reflection but I have never used it to solve a problem"
tags: [aspnet core, reflection]
categories: [Blogging]
excerpt_image: https://raw.githubusercontent.com/KamRon-67/jrdev.io/master/assets/img/reflections_201617.jpg
---

Like most people during covid, I missed my coding groups!!! I use to belong to three or four groups. While they are still around none of my local branches decided to go virtual. So I expanded my options and dropped into any open C# groups. I enjoyed it. Excluding my main coding group I had not been around other non-work devs. We would have talks about bout complex things. One thing that kept popping up was refection. Most of the people I am around are more experienced than me. So every time it was brought up we would just gloss over reflection and we would all laugh about why it was used. While I was laughing, I would get lost in what came next. I understood what reflection was as a concept. Never seeing it in action or using it, limited that understanding. So enough back story this is the what is reflection post.

Reflection is used when we want to operate on an object during runtime. I like to think of it as creating objects backward. Reflection is a meta-programming feature that many programming languages support. You can write code that inspects other code, the same system, or even itself. It sometimes can modify the behavior of that code or methods during runtime. That is my more technical definition. I was able to come up with that after I figured out my simple example.


![A tree over looking the water](https://raw.githubusercontent.com/KamRon-67/jrdev.io/master/assets/img/reflections_201617.jpg "TA tree over looking the water")

In reflection, we are interested in the [types](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/) and [members](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/members). Typically in a program, you create objects, call functions, set values for the properties of this class. When using reflection, things are different you have to construct what you are going to reflect on. We are not interested in the instance or its values, properties but we are interested in its type itself. What properties and methods it has. We can call those methods at run time.

Looking at the [Microsoft doc](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection), all I have to do is use the GetType method and bam magic!! That was not enough for me. Seriously there is another [link](https://docs.microsoft.com/en-us/dotnet/framework/reflection-and-codedom/reflection) with more information. The way it is used and when to use it. Were still things that confused me. So I made a short and useless program. To use reflection to produce an action.

```csharp
var query = from m in typeof(string).GetMethods()
			where m.IsStatic == true
			orderby m.Name
			group m by m.Name into g
			orderby g.Count()
			select new { Name = g.Key,
			Overloads = g.Count()};

			foreach(var item in query)
			{
				Console.WriteLine(item);
			}
```
Here we drilled into the static string class to see how many overloads each method has then ordered them first by the number of overloads then alphabetically. This was a simple example of reflection. The GetMethods method shuffles its order each call. You would not want to depend on it without a constraint. Let's look at more of a breakdown. A real-world use of reflection would be a unit testing framework. We are not going to create one, but look at how it works in an isolated point. The full code is [here](https://github.com/KamRon-67/reflectionProj/blob/Master/Program.cs) If you want to see how to implement a full framework this [blog here](https://devblogs.microsoft.com/dotnet/show-dotnet-build-your-own-unit-test-platform-the-true-story-of-net-nanoframework/) is for you.

```csharp
Type personType = typeof(Person);

var properties = personType.GetProperties();

foreach (var item in properties)
{
	Console.WriteLine($"Property: Type: {item.PropertyType.Name}
			| Name: {item.Name}");
}

Person person = new Person();

var methods = personType.GetMethods();
foreach (var item in methods)
{
	Console.WriteLine($"Method: Type: {item.ReturnType.Name}
			| Name {item.Name}");

	if (item.Name == "Print")
	{
		item.Invoke(person, null);
	}
}
	Console.WriteLine("-------------------------------");
	AttributeTest(typeof(Person));

	Console.ReadKey();
```

Let's start with the main method. We use the [Type](https://docs.microsoft.com/en-us/dotnet/api/system.type?view=net-5.0) class and the [typeof](https://docs.microsoft.com/en-us/dotnet/api/system.type?view=net-5.0) method. This will give us the ability to create an instance of our person type. We use the GetMethods method on that instance, now we will get all of the public methods of Person.

Here we create a person class and a custom [attribute](https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/attributes). By giving it a parameter later we can use set values during our refection process. Putting the "Attribute" in the class name seems to be the standard. I am not a fan as it is a little confusing.

```csharp
public class RunMethodAttribute : Attribute
{
	public int Count { get; set; }
}
```

This is our simple class we will use as a stand in object.

```csharp
public class Person
{
	public string FirstName { get; set; }
	public string LastName { get; set; }
	public string Phone { get; set; }
	public int ZipCode { get; set; }
}
```

Now we are using our custom attribute, we can set values now and use them later in our reflection call. We will use ours in a loop. You can pass many things to an attribute but you are limited the doc will cover the do's and don'ts.

```csharp
[RunMethod(Count = 3)]
public void Print()
{
	Console.WriteLine($"{FirstName} {LastName}");
}

[RunMethod(Count = 3)]
public void TestMethod()
{
	Console.WriteLine("Hello from TestMethod");
}

public void Move(int newZipCode)
{
	ZipCode = newZipCode;
	Console.WriteLine($"{FirstName} {LastName} has been moved to {newZipCode}");
}

[RunMethod(Count = 1)]
public void SayHi()
{
	Console.WriteLine($"Hi {FirstName}");
}
```

Like above we are doing something similar in our linq statement. We are using the type class to get all of the methods of our passed in value of person. Because we are using the type class we can use all the type methods. We are using [GetMethods](https://docs.microsoft.com/en-us/dotnet/api/system.type.getmethods?view=net-5.0) and [GetCustomAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.attribute.getcustomattribute?view=net-5.0) with our favorite method typeof. Now we are only getting the methods that are of or custom attribute type RunMethodAttribute. With the [Activator](https://docs.microsoft.com/en-us/dotnet/api/system.activator?view=net-5.0) class I can be lazy and just use the CreateInstance method. Now we have what I would call our shell object (that is not an official term). Using the type class we are forced to use casting while looping. We can use our attributes to power our loop logic.

```csharp
static void AttributeTest(Type type)
{
	// Get the methods
	var allMethods = type.GetMethods();
	var methodsWithAttribute = allMethods.Where(m => m.GetCustomAttribute(typeof(RunMethodAttribute)) != null);

	var obj = Activator.CreateInstance(type);

	foreach (var item in methodsWithAttribute)
	{
		var attribute = (RunMethodAttribute)item.GetCustomAttribute(typeof(RunMethodAttribute));
		Console.WriteLine($"{item.Name} run for {attribute.Count} times");
		for (int i = 0; i < attribute.Count; i++)
		{
			item.Invoke(obj, null);
		}
	}
}
```

For me looking at how something is used in the real world is helpful for me to know other times I may need it. For the longest, I was not able to say when to correctly using refection. I still can't but I know of real-world moments like this, EF Core and the DBContext class, or loading DLLs using Assembly.LoadFile. That helps me "reflect" on the situation I am currently working on. Then know if reflection should be used.