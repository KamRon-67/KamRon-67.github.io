---
title: "Working With WebApplicationFactory"
description: "Working with the WebApplicationFactory class to create tests"
tags: [TDD, Testing, Testing Framework, C#, WebApplicationFactory, Xunit]
categories: [Blogging]
---

Throughout my career, I have seen the testing pyramid. I was under the impression that one should create more unit tests than integration tests with a dash of UI testing. The first part of my career. I was an automation developer. So, I only did UI testing. After moving back to the formal development side. To my shock many of the places I worked. Testing was not integral to the dev process. Personally, this burned me and added unnecessary stress to my life.

That was then, now. I test software in ways that help me sleep at night. Later when a bug is found or we need to extend this feature. I noticed zero stress on my side. I can make the changes quickly and possibly improve the logic. The requirements are locked in. Finding myself on the backend of things more than the front end. Integration tests gives me more value as I am creating several moving parts. 

When I first started out testing my software I would tend to blur the lines between unit tests and integration tests. My unit tests may touch a few parts of the system and many would not consider them unit tests any longer. There would not be any calls to a database or service. I would mock the responses. This blurring was a compromise between testing at all and working with a client's ci cd process.

This is when I found out about the WebApplicationFactory class. I was late to the party and probably working on a legacy application. Legacy applications are sometimes my bread and butter. Staying current on new ways to test is best. Then I stumbled onto the WebApplicationFactory.

## How I see it

> To me, the WebApplicationFactory class is just an in-memory application that can handle HttpClient, in-memory DB's it can do more tho.

I am only able to use this with newer projects .net 3 plus. For me, that is not a problem as I want to distance myself from dotnet framework.  One can configure test-specific implementations of services. Or override the default behavior in the WebApplicationFactory instance. This opens a TDD lane for me. 

Here is a simple example of using WebApplicationFactory for testing an ASP.NET Core MVC application [**(Full app here)**](https://github.com/DamianEdwards/MinimalApiPlayground):


```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace MinimalApiPlayground.Tests;

internal class PlaygroundApplication : WebApplicationFactory<Program>
{
    private readonly string _environment;

    public PlaygroundApplication(string environment = "Development")
    {
        _environment = environment;
    }

    protected override IHost CreateHost(IHostBuilder builder)
    {
        builder.UseEnvironment(_environment);

        // Add mock/test services to the builder here
        builder.ConfigureServices(services =>
        {
            services.AddScoped(sp =>
            {
                // Replace SQLite with in-memory database for tests
                return new DbContextOptionsBuilder<TodoDb>()
                .UseInMemoryDatabase("Tests")
                .UseApplicationServiceProvider(sp)
                .Options;
            });
        });

        return base.CreateHost(builder);
    }
}

```

```csharp
using System.Net;
using System.Net.Http.Headers;
using Xunit;

namespace MinimalApiPlayground.Tests;

public class TodoApi
{
    private static readonly string _validTodosJsonFileName = "todos-valid.json";
    private static readonly string _invalidTodosJsonFileName = "todos-invalid.json";

    [Fact]
    public async Task POST_FromFile_Valid_Responds_Created()
    {
        await using var application = new PlaygroundApplication();

        using var formContent = new MultipartFormDataContent();
        using var fileContent = new StreamContent(File.OpenRead(_validTodosJsonFileName));
        fileContent.Headers.ContentType = MediaTypeHeaderValue.Parse("application/json");
        formContent.Add(fileContent, "todosFile", _validTodosJsonFileName);

        using var client = application.CreateClient();
        using var response = await client.PostAsync("/todos/fromfile", formContent);
        var responseBody = await response.Content.ReadAsStringAsync();

        Assert.Equal(HttpStatusCode.Created, response.StatusCode);
        Assert.NotNull(response.Headers.Location);
        Assert.Matches("My Todo from a file", responseBody);
        Assert.Matches("Another Todo from a file", responseBody);
    }

    [Fact]
    public async Task POST_FromFile_Invalid_Responds_BadRequest()
    {
        await using var application = new PlaygroundApplication();

        using var formContent = new MultipartFormDataContent();
        using var fileContent = new StreamContent(File.OpenRead(_invalidTodosJsonFileName));
        fileContent.Headers.ContentType = MediaTypeHeaderValue.Parse("application/json");
        formContent.Add(fileContent, "todosFile", _invalidTodosJsonFileName);

        using var client = application.CreateClient();
        using var response = await client.PostAsync("/todos/fromfile", formContent);
        var responseBody = await response.Content.ReadAsStringAsync();

        Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
        Assert.Equal(response.Content.Headers.ContentType, MediaTypeHeaderValue.Parse("application/problem+json"));
        Assert.Matches("\\[1\\].Title", responseBody);
        Assert.Matches("The Title field is required", responseBody);
    }
}

```

Well after looking at the miniamal api's I had one question. WebApplicationFactory<T> expects an entry-point-class. What do I give it?  That was a common question and the answer is [here](https://stackoverflow.com/questions/69058176/how-to-use-webapplicationfactory-in-net6-without-speakable-entry-point). There was also an answer about how to set this up inconjuction with xunit's IClassFixture. 

WebApplicationFactory is a valuable tool for integration testing ASP.NET Core applications. It provides a complete testing environment, including a deployed app and an in-memory server, making it much easier to set up, and run tests.  