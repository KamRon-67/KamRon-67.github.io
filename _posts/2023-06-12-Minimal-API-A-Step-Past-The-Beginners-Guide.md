---
title: "Minimal APIs in C#: A Step Past the Beginner’s Guide"
tags: [C#, Minimal, Python, JS]
categories: [Blogging, API]
---

Introduction:
Being in the C# space for over six plus years. You get in the habit of doing things the dotnet way. I personally like that dotnet is very vocal in patterns and how to structure things. With the release of dotnet 6 we have access to Minimal APIs. While C# pays my bills and is my personal favorite. (followed by swift it has been growing on me) I have used other languages. When I first saw Minimal APIs my brain just went to similar Express and Flask, example below. Both of those frameworks have been out for years. I have used both to make some cool things. So my first thought was “How do I increase the  cyclomatic complexity?” 

Hello World Examples:


```js
const express = require("express");
const app = express();

app.get("/", (req, res) => {
  res.send("Hello, World!");
});

app.listen(3000);
```

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello, World!");

app.Run();
```

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
  return "Hello, World!"

if __name__ == "__main__":
  app.run()
```

What are Minimal APIs?
Minimal APIs are a simplified way to define and implement HTTP endpoints in C#. There is way more to it especially when you get into how top level statements work in all of this. I don’t use the other part of the definition from the docs, when explaining Minimal APIs. The documentation is not incorrect, but I am thinking about impact vs. intent. The intent or purpose of why this feature was released and the impact is what we achieve with it as a community. I don’t have an answer for that yet. Minimal API’s are not going to replace what we are doing now and may projects may start off as Minimal APIs then grow into a normal api project. 


Setting Up the Environment:
To begin, you'll need to have the following prerequisites installed on your machine:
1. .NET 6.0 SDK or later
2. An Integrated Development Environment (IDE) such as Visual Studio 2022 or Visual Studio Code.

Creating a Minimal API Project:
1. Open your preferred IDE and create a new C# project.
2. Select the ASP.NET Core Web Application template.
3. Choose the Minimal API template to create a project specifically for minimal APIs.

Defining Endpoints:
With a minimal API project created, it's time to define your endpoints. Let's start by creating a simple "Hello, World!" endpoint.

1. Open the `Program.cs` file, modify it to look like the following:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<DataContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

app.UseHttpsRedirection();

app.MapGet("/car", async (DataContext context) =>
    await context.Cars.ToListAsync());

app.MapGet("/car/{id}", async (DataContext context, int id) => 
    await context.Cars.FindAsync(id) is Cars car ?
    Results.Ok(car) :
    Results.NotFound("Sorry, car not found."));
```

2. Note: I am not going over how to set up ef core and run a migration. I don’t want to deviate from the core point too much. Even in the example I am using sqlite just to get some data to play with.


```csharp
    public class DataContext : DbContext
    {
        public DataContext(DbContextOptions<DataContext> options) : base(options)
        {

        }

        public DbSet<Cars> Cars => Set<Cars>();
    }
```

```csharp
    public class Cars
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Make { get; set; } = string.Empty;
        public string Year { get; set; } = string.Empty;
        public int Price { get; set; }
    }

```

Problem: 
This is one of those great clean examples. I don’t live in a clean world. How would one structure this to handle a non-trivial amount of API calls? Small projects, the requirements are going to change, there will be a few “Can we sneak this in real quickly?”.

A Possible solution: Removing things from the Program file. 

I would start by removing the IServiceCollection items. This is something that has the possibility to grow, right now I am passing the DataContext around and that is a big no no. If I was to just add the repository pattern this section will grow. 

So something like this:
```builder.Services.AddScopped<ICarRepository, CarRepository>();``` 
This is just one, this could also grow depending on our needs. I would hate for this to be in the Project.cs file with the API calls. 

First update: 

```csharp
var builder = WebApplication.CreateBuilder(args);

RegisterServices(builder.Services);

var app = builder.Build();

app.UseHttpsRedirection();

app.MapGet("/car", async (DataContext context) =>
    await context.Cars.ToListAsync());

app.MapGet("/car/{id}", async (DataContext context, int id) => 
    await context.Cars.FindAsync(id) is Cars car ?
    Results.Ok(car) :
    Results.NotFound("Sorry, car not found."));

app.Run();

 
void RegisterServices(IServiceCollection services)
{
    services.AddScopped<ICarRepository, CarRepository>();
    services.AddDbContext<DataContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
}
```

This is a method but this could be its own class, whatever works for us. The goal is just to remove some of the set up code from the Program.cs file. 

Fun Fact: The code below would have worked as well. I just made it verbose so I could move it around more if I wanted too. If you have not used top level statements this might look odd since the builder is not passed in. The way top level statements work, it is behind the scenes compiling a main and a start up class for you. So long in Program.cs builder is in scope wherever you use it.  

```csharp
void RegisterServices()
{
    builder.Services.AddScopped<ICarRepository, CarRepository>();
    builder.Services.AddDbContext<DataContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
}
```

I am liking what we have going on here.

I would like to pull out those map calls out of the program file. 

```csharp
    public class ClientApi
    {
        public ClientApi()
        {
        }

        public void Register(WebApplication app)
        {
            app.MapGet("/", () => "Hello, World!");

            app.MapGet("/hello/{name}", (string name) => $"Hello, {name}!");

            app.MapGet("/car", async (DataContext context) =>
                await context.Cars.ToListAsync());

            app.MapGet("/car/{id}", async (DataContext context, int id) =>
                await context.Cars.FindAsync(id) is Cars car ?
                Results.Ok(car) :
                Results.NotFound("Sorry, car not found."));
        }
    }

```

After adding the ClientApi class I am fully able to reduce the Program.cs class to this. 

```csharp
using MinimalApi;

var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.UseHttpsRedirection();

new ClientApi().Register(app);

app.Run();

void RegisterServices(IServiceCollection services)
{
    services.AddDbContext<DataContext>(options =>
    options.UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
}

```

Conclusion:
We could do more, we could even use reflection to get our mapping. After playing with Minimal APIs my biggest takeaway is the new freedom you get to do things your own ways. Remember, this blog post only scratches the surface of what minimal APIs can do. Continue exploring the ASP.NET Core documentation and examples to dive deeper into this exciting new feature.