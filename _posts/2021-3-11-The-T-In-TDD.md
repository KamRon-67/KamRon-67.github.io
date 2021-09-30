---
title: "The T In TDD"
description: "How to set up tests to do TDD"
tags: [TDD, Testing, Testing Framework, C#, NUnit]
categories: [Blogging]
excerpt_image: https://raw.githubusercontent.com/KamRon-67/jrdev.io/master/assets/img/orangeFlower.jpg
---
 
By this point. If you have read some of my older blog postings. You know I am a part of a programming group that is big on TDD and the DDD paradigm. My mind is all over the place and I learn 5 things at one time. During this storm of random learning. I picked up TDD while reading "Working Effectively With Legacy Code" by Michael C. Feathers.
The main focus is clearly stated in the title "Test" Driven Development.
 
If you are going to do TDD you will need to know how to test software. I did not learn about testing software in college. The many jobs I have worked at. Only a few of them had tests at all. Only when I became an automation developer was I forced into the world of testing. Even then I was still able to ignore many of the great benefits it offers.
 
Here we are going to go over a unit testing framework. There are a lot of choices like XUnit (my personal favorite), NUnit, pytest for python (My second favorite), and Jest for javascript. In our example, we will cover NUnit. While XUnit is my favorite I need to use NUnit for work so here we go!

This is more of a "how to structure" your tests. We will not go over "how" to test. Just like everything else in software, there are layers to testing. Picking a testing framework, assertion framework, and maybe a mocking framework. There could be more depending on your needs.

Personally, starting something is hard. I always get hung up on best practices and structure. This post will not jump into best practices as that is a post all of its own but just lightly touch the options you have in a testing framework. This should get you started on your TDD journey.

Many frameworks follow an attribute pattern. They use attributes to dictate functionality. I will cover the NUnit style. XUnit shares a history with NUnit and both are extremely popular. There are reasons to pick one over the other but that again is a post by itself. Some frameworks won't follow the same attribute style per-se such as PyTest for python.  The spirit of it is still there as Pytest will do something like ```@pytest.fixture``` above a method.

There are many attributes available in a test framework. Some of the most common are fixtures, test, set up, tear down, ignore, and category. There are more options but we will narrow our focus. There is a long list and you can find all of the NUnit attributes [here](https://docs.nunit.org/articles/nunit/writing-tests/attributes.html).

Let us start with fixtures. They are to ensure that there is a known environment so test results are repeatable. In plain English, it takes a lot of work to set up a test then reset everything after the test has run. We plan to execute this test many times. So we want an easy way of making sure this process is repeatable. Below is a simple example

```csharp
    [TestFixture]
    public class Tests
    {
        // ...
    }
```

There are a few ways you can use to structure your fixtures depending on your needs. One of those patterns is the abstract test pattern. This is a pattern for when you want to test several implementations. You could write a test for each implementation or you could create an abstract class containing all of the tests. Then using a factory pattern you implement each concrete test.

```csharp
    namespace UnitTestProject1
    {
        public class TestArrayList : BaseTestStringList
        {
            protected override IList CreateList()
            {
                return new ArrayList();
            }
        }

        public class TestStringCollection : BaseTestStringList
        {
            protected override IList CreateList()
            {
                return new StringCollection();
            }
        }

        [TestFixture]
        public abstract class BaseTestStringList
        {
            private IList list;

            [SetUp]
            public void SetUp()
            {
                this.list = CreateList();
            }

            protected abstract IList CreateList();
 
            [Test]
            public void TestAdd()
            {
                object item = Guid.NewGuid().ToString();
                int beforeCount = this.list.Count;
                this.list.Add(item);
                Assert.AreEqual(beforeCount + 1, this.list.Count);
            }

            [Test]
            public void TestContains()
            {
                object item = Guid.NewGuid().ToString();
                int beforeCount = this.list.Count;
                this.list.Add(item);
                Assert.IsTrue(this.list.Contains(item));
            }

            [Test]
            public void TestClear()
            {
                object item = Guid.NewGuid().ToString();
                this.list.Add(item);
                this.list.Clear();
                Assert.AreEqual(0, this.list.Count);
                Assert.IsFalse(this.list.Contains(item));
            }
        }
    }
```
[Source](https://weblogs.asp.net/nunitaddin/134151)


This is just one way of using inheritance, generics, or parameterization. You can create a pattern that works for you. Depending on the language you choose to use. There may be pattern limitations or you will have to structure them differently. When moving from a c# automation project to a python project. I had to rethink a lot of what I did. Eventually we threw away our page object pattern. For a more dynamic data-driven approach using JSON files.

The test attribute just marks a test case. More specifically this attribute lets the test runner know this method should be run.

```csharp
 [TestFixture]
  public class SuccessTests
  {
    // A simple test
    [Test]
    public void Add()
    { /* ... */ }
    ....
  }
```

So there are two types of teardowns. We have the OneTimeTearDown and the TearDown. The OneTimeTearDown attribute is set on methods that run once after executing all the tests in a fixture. The TearDown, unlike the OneTimeTearDown, this attribute will trigger after the test method. OneTimeSetup is best used for costly steps like creating a database. It is more on the integration test side of things. Like everything in software, the tool is at your disposal.

```csharp
namespace NUnit.Tests
{

  [TestFixture]
  public class SuccessTests
  {
    [OneTimeSetUp]
    public void Init()
    { /* ... */ }

    [OneTimeTearDown]
    public void Cleanup()
    { /* ... */ }

    [Test]
    public void Add()
    { /* ... */ }
  }
}
```

The OneTimeSetup and SetUp attributes work the same way in reverse.

The Ignore attribute will skip any test it is applied to.

This is a good starting point for working with a test framework. There is more to testing than this. Remember this is more of a "how to structure" your tests. So we did not go over how to test at all. Eventually, you would want to assert that the data you get is the data you expect.

When I was introduced to testing I was not aware of any framework options patterns or ways to structure my tests. Or even how to test.
In the dot net space, you can unit test razor pages, controllers, and middleware. Testing things like this can help you maintain the integrity of your project as it grows or if you need to do a refactor down the line.

Say you are moving from an ORM to a more traditional database setup. For whatever reason. Having tests fail while you are working on the migration. Everything is fresh and you are in the same headspace. Is a lot better than the same bug popping up 6 months later. By then you are on to something else.

