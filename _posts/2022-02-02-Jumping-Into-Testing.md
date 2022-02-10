---
title: "Using Tests To Speed up FeedBack Loop"
mermaid: true
tags: [C#, Unit Testing, Testing, Test Doubles, Mocks, London School, Detroit School, Collaborators, Dependencies]
categories: [blogging, testing]

---

## Why Do I Even Care to Test?
I have worked in a few code bases that only had tests at the center layers of the application. The team would rarely touch that code plus there is a good chance the test was stale. Any massive feature would lead to so much manual regression. We would lose up to a week of the sprint. Coming from a Q.A. background there is a process to do a manual regression. There are patterns and software that show steps as test cases. Using the software you would know what was done and you could have a pretty good estimate of how long you are doing regressions.



## Doing Things the Hard Way
Fast forward a few years. I moved over to the dev side, found myself doing two or three sprints then a massive “regression”. To be fair I worked in smaller companies so there were no Q.A. Team, C.I., or C.D. processes. No End 2 End, integration, or unit tests. For the most part, this did not bother me. I had figured this is just what you did as a software dev. After the epic was done you just spent three weeks fixing the side effects of the new feature. To be clear we were not allotted three weeks. We would just be given three days before new work was given. It would just take that long doing two things at once.

```mermaid
flowchart TD
    A[Current Task] --> B{Is there a bug?};
    B -->|Yes| C[Work on Bug];
    C --> D[Eye Ball It];
    D --> B;
    B ---->|No| E[Next task bug combo];
 ```
 > This was the unspoken sub workflow. Possibly error-prone and slow. Bugs just pop up.


At this point in my career, I did not know much about unit testing. Other devs I worked with would say unit testing is a waste of time. Spoiler alert, it is not. One time the pressure was on us. I had two views that were full of similar business logic that needed to be worked on in parallel. Yes, I am aware views should not contain business logic but at this company. As a team, we could not spell the word "refactor". So we just added code on top of code. No tests, just manual regressions. To verify we did not break existing functionality. 

## My Breaking Point
Each view had a jquery powered dropdown that would populate dynamic dropdowns based on an ajax call. Depending on the combination of drop dropdowns. There could be various page combinations in the same section. As you can start to see, this is just a problem waiting to happen. Not shortly after I was “done” not only did I have a few bugs to fix. The requirements changed. On paper the logic was clear, both views should act the same. The problem was both views had been created by two different consultants with two different coding styles. With zero communication between each other. One view used custom templating logic and another used a very complex nested loop with a poor naming convention. The requirements for this section changed multiple times. This was the first time I wanted tests. Both views were broken before we added code. This section of code gave me the blues. To test any of your changes you had to do a long manual process. With this long feedback loop verifying any changes was that much harder. This project put me on my testing journey. Before then I had done years of dev work without writing any tests for my work. The long feedback loop was burning me out. Fast forward to 2022 I pretty much use integration tests whenever possible.

So later I got a mentor and started down my padawan journey again. I was told to do kata's and use unit tests "File logger kata [class1](https://githistory.xyz/KamRon-67/file-logger-kata/blob/master/FileLogger/FileLogger_Log.cs) [class2](https://githistory.xyz/KamRon-67/file-logger-kata/blob/master/FileLogger/FileLogger.cs) to keep my code structured. Now I was face to face with my unit testing ignorance. I then decided to do the zero to hero route. When I learn new things I have to learn everything so I know what to ignore later. So why even unit test? To get a workflow like this. Something like this would have told me when my changes were breaking my views.

```mermaid
flowchart TD
    A[Build Process] --> B{Test passing?};
    B -->|No| C[Work on Bug];
    C --> D[Run test];
    D --> B;
    B ---->|yes| E[Nothing/Next Task];
 ```
 > Here the system checks for breaking changes if set up correctly. Bugs come to you! 

## compiling test knowledge
Well, code tends to deteriorate each time it is changed. You don’t know if you are introducing new problems. Even at this moment, unit tests seemed like a magical thing that would keep my code clean. I was not aware that tests only work in one direction. If you can not unit test the code, it is low quality. In the same breath. Just because you can unit test the code doesn't mean it is quality code.
    
Ok so now I am a tester there are two paths I could go down classical and London. Both styles verify a piece of code, both do it quickly and in an isolated manner. Now what isolated means is where the paths start to differ. London style you want to focus on the class under test as much as possible. Any dependencies you would want to mock or use test doubles.  

```csharp
public class Engine 
{
    private CarPart _cylinder;
    private readonly IgpsRepository _gpsRepository;
    private readonly ILocationServices _locationservices;

    public engine()
    {
    }

    public engine(IgpsRepository gpsRepository, ILocationServices _locationservices, CarPar cylinder)
    {
        _cylinder = cylinder;
    }
         //.. Do things

    public CarAction drive(Speed speed)
    {
        // do things
        return carAction;
    }
}
```
    

```csharp
// ... In a test class at the top part 
private readonly IgpsRepository _gpsRepository = Substitute.For<IgpsRepository>();
private readonly ILocationServices _locationservices = Substitute.For<ILocationServices>();

// ... In a test class
[Fact]
public void CoolTestName()
{
    // simple test double example
    var part = new CarPart(); <-- test double
    var _sut = new Engine(part); 
    //... Test and things
}

[Fact]
public void CoolTestName()
{
    // Data for mock
    var latitude = new Latitude
    {
        Latitude = 4,
        Longitude = 5
    }

    var locaton = new Location
    {
        State = "state"    
    }

     _gpsRepository.GetLatitude(4,5).Returns(latitude));
     _locationservices.GetState("state").Returns(locaton);
    // Using mocks to make test doubles example
    var part = new CarPart(); // <-- test double
    var speed = new Speed(55);
    var _sut = new Engine(_gpsRepository, _locationservices, part); // <-- using mocks
    var action = _sut.drive(speed);
    //... Test and things
    
    Assert.NotNull(result);
}
```
> There are many types of doubles Dummy, Fake, Stub, Spy and Mock to name a few.

The goal here is to separate the classes heavier from any external influence. In the classical approach, we just want to do all of the unit tests in an isolated manner. We want all of the tests to avoid talking to any shared state. The classical style is looser, a unit could be a single class or a set of classes. Long as you use test doubles for any shared dependencies. After getting an understanding of my options I see the London style is not my cup of tea. 

```csharp
public interface ICalculator
{
    int Add(int a, int b);
    string Mode { get; set; }
    event EventHandler PoweringUp;
}

// In test file 
[Fact]
public void CoolTestName()
{
    // Setting up the mock
    calculator = Substitute.For<ICalculator>();
}
```
> This is an example from the nsubstitute docs.

In the past, I worked in codebases that had little to no interfaces. Using mocks would result in me creating interfaces and proxy classes to get things under test. This has to be done because mocking classes is [bad practice](https://stackoverflow.com/questions/1595166/why-is-it-so-bad-to-mock-classes). I will use a mock but it is a last resort. Working in messy codebases. Getting anything under test is already a challenge, I need all of the flexibility I can get. If you're open to reading PHP this was a cool [little guide to testing](https://github.com/sarven/unit-testing-tips#introduction)

Test doubles are just the start of the dive into the talk of dependencies in testing. Understanding collaborators and dependencies allows you to avoid pitfalls when making tests. Just like with test doubles, there are many types of dependencies. 
 

## Flavors of Dependencies
The first thing I had to wrap my head around is what dependency is. When you read something it may not click till you use it, or do some more digging. Shared, private, and out-of-process dependencies came up the most. 
What is a collaborator? Anything mutable. Classes like the dbContext would be a collaborator because it is providing access to the database. The DB is a shared dependency. Value types or objects can be dependencies as well. A shared dependency is a dependency used between tests and can affect each other's outcomes. A private dependency is not shared. An out of process dependency runs outside the applications execution process. This is a rabbit hole of what's what information.

