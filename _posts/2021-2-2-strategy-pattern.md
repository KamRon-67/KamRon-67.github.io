---
title: "Strategy Pattern"
description: "Strategy broken down"
tags: [Strategy Pattern, Strategy, gang of 4, C#, Asp.net]
categories: [Blogging]
excerpt_image: https://raw.githubusercontent.com/KamRon-67/jrdev.io/master/assets/img/flower3.jpeg
---

I joined this [group](https://devbetter.com/) to improve my programming skills and understanding of software concepts. In this group, there is a book list. One of those recommended books is Head First Design Patterns. In this book, the strategy pattern is one of the first you will encounter. Using Head First's definition, this pattern is a family of algorithms (business logic) that encapsulates each algorithm and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it. I like the high-level explanation. I also like things to be simplified.

The Subreddit explains it like I am five, which is a goldmine for me. It breaks down complex topics to gain a better understanding. It helps me personally. Let's simplify this pattern. The first time I ran across this software pattern, my eyes glossed over. I was not a fan of the example from the book. It gets the job done, but I made my own. I made two examples; a simple and more complete version.

In my first example, we will think of a website login portal. With portals, you can log in using an email, phone, or social media account. Because this is a simple example, we will stick with those three options.

We have different classes with the needed logic to login to the website. Each class implements the same interface. This interface tells each class to make a method. The method signature will be the same in each class, but the business logic will vary from class to class. You can find the full code [here](https://github.com/KamRon-67/Simple_Strategy/blob/master/Simple_Strategy/Program.cs)

```csharp
public class LoginUsingEmail : IAsyncRequestStrategy
  {
    public AsyncResponse SendRequest(String url)
    {
        var asyncResponse = new AsyncResponse();
        Console.WriteLine("Sent login request using email");
        return asyncResponse;
    }
  }

    // Login Using Phone
public class LoginUsingPhone : IAsyncRequestStrategy
  {
       public AsyncResponse SendRequest(String url)
    {
        var asyncResponse = new AsyncResponse();
        Console.WriteLine("Sent login request using phone");
        return asyncResponse;
    }
  }

    // Login Using Social Media
public class LoginUsingSocialMedia : IAsyncRequestStrategy
  {
    public AsyncResponse SendRequest(String url)
    {
        var asyncResponse = new AsyncResponse();
        Console.WriteLine("Sent login request using social media");
        return asyncResponse;
    }
  }
```

This was the first major take away for me. Using interfaces lets us decouple the logic. In the main method, the web application class is passed in a "How to login" value via an enum. From there, the class at runtime picks the needed class to log in. In this example, the Send Request methods are not doing anything.
I was moving in the correct direction with my first example trying to follow the teachings from the Head First book "Design Principle- Program to an interface, not an implementation" and "Identify the aspects of your application that vary and separate them from what stays the same."

This is the end of the simple example, but it has its [issues](https://stackoverflow.com/questions/3834091/strategy-pattern-with-no-switch-statements). In my example, the switch statements are vague and super high level. With real logic, they may no longer be as nice as they are now. There could be variations or new cases, and things can start to get bad. We will want to avoid switch creep.

```csharp
public AsyncResponse SendAsyncRequestToServer(string url)
        {
            IAsyncRequestStrategy asyncRequestStrategy;
            AsyncResponse asyncResponse = null;
            switch (_loginType)
            {
                case LoginType.Email:
                      asyncRequestStrategy = new LoginUsingEmail();
                      return asyncRequestStrategy.SendRequest(url);

                case LoginType.Phone:
                      asyncRequestStrategy = new LoginUsingPhone();
                       return asyncRequestStrategy.SendRequest(url);
                case LoginType.SocialMediaAccount:
                    asyncRequestStrategy = new LoginUsingSocialMedia();
                    return asyncRequestStrategy.SendRequest(url);
            }

            return asyncResponse;
        }
```

Nasty code could get placed into this section, and it could become a pain point.

Now this version would be more in line with the pattern from the book.

```csharp
using System;

namespace Fixed_Strategy
{
    class MainApp
    {
        static void Main()
        {
            Login login;

            // Three contexts following different strategies

            login = new Login(new LoginUsingEmail());
            login.LoginWebsite();

            login = new Login(new LoginUsingPhone());
            login.LoginWebsite();

            login = new Login(new LoginUsingSocialMedia());
            login.LoginWebsite();

            // Wait for user

            Console.ReadKey();
        }
    }

    abstract class LoginStrategy
    {
        public abstract void SendLoginRequest();
    }

    class LoginUsingEmail : LoginStrategy
    {
        public override void SendLoginRequest()
        {
            Console.WriteLine(
              "Logging in using LoginUsingEmail");
        }
    }

    class LoginUsingPhone : LoginStrategy
    {
        public override void SendLoginRequest()
        {
            Console.WriteLine(
              "Logging in using LoginUsingPhone");
        }
    }

    class LoginUsingSocialMedia : LoginStrategy
    {
        public override void SendLoginRequest()
        {
            Console.WriteLine(
              "Logging in using LoginUsingSocialMedia");
        }
    }

    class Login
    {
        private LoginStrategy _loginStrategy;

        // Constructor

        public Login(LoginStrategy strategy)
        {
            this._loginStrategy = strategy;
        }

        public void LoginWebsite()
        {
            _loginStrategy.SendLoginRequest();
        }
    }
}
```

This is also simple, but without the switch statement, the main smell is removed. All logic is in a class, and to add any new rules, we would simply add a class. If you know dependency injection, you pretty much know Strategy Pattern!
