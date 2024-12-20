---
title: "Building AI Powered Applications."
description: "We are creating a local AI using aspire"
tags: [DIP, AI, C#, Code]
categories: [Blogging]
---

Artificial Intelligence (AI) is transforming the way developers create applications. It’s up to us to adapt and grow with it. With frameworks like .NET Aspire, building local AI-powered solutions has never been more accessible. Having a local environment to prototype innovative solutions is essential—whether you're working on a chatbot, processing images, or analyzing text. Plus, with Aspire, you can seamlessly scale and deploy to the cloud when needed.

In this guide, we’ll walk through the steps to create a local AI-powered application using .NET Aspire. To demonstrate its potential, we’ll build a chatbot and showcase how to integrate AI libraries into your .NET project.

---

## **Prerequisites**

Before we start, ensure you have the following:

- [.NET 9 SDK](https://dotnet.microsoft.com/download) installed on your machine.
- Knowledge of C# and Blazor.
- [Aspire](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/setup-tooling?tabs=linux&pivots=dotnet-cli) set up and tooling

---

## **Step 1: Setting Up Your .NET Aspire Project**

### Install Dependencies

For this demo, we’ll use CLI templates, which you can find [here](https://learn.microsoft.com/en-us/dotnet/aspire/fundamentals/aspire-sdk-templates?pivots=dotnet-cli#install-the-net-aspire-templates)
Run the following commands to get started:

```
dotnet new install Aspire.ProjectTemplates
```

```
dotnet new list aspire
```

Start by creating a new Aspire project:

```bash
mkdir LocalAIChatbot
cd LocalAIChatbot
dotnet new blazorserver -n YourProjectsName cd YourProjectsName
```

Next, install the required AI libraries:

AppHost:
- `dotnet add package CommunityToolkit.Aspire.Hosting.Ollama --version 9.1.0`

Web:
 - `dotnet add package Azure.AI.Inference --prerelease`
 - `dotnet add package Microsoft.Extensions.AI --version 9.0.1-preview.1.24570.5`
 - `dotnet add package Microsoft.Extensions.AI.Ollama --version 9.0.1-preview.1.24570.5`
 - `dotnet add package Microsoft.Extensions.AI.OpenAI --version 9.0.1-preview.1.24570.5`
 - `dotnet add package CommunityToolkit.Aspire.Hosting.Ollama --version 9.1.0`

(The )
### Add Configuration

Create an `appsettings Development.json` for AI Key data in the Web project:

```json
{  
  "ConnectionStrings": {  
    "ollama": "http://localhost:11434",  
    "llama3": "llama3"  
  },  
  "DetailedErrors": true,  
  "Logging": {  
    "LogLevel": {  
      "Default": "Information",  
      "Microsoft.AspNetCore": "Warning"  
    }  
  }  
}
```

You can adjust the port if necessary, and we’ll cover this later.

---

## **Step 2: Adding A little Code**

### AppHost/Program.cs 
```csharp
var builder = DistributedApplication.CreateBuilder(args);  
  
var cache = builder.AddRedis("cache");  
  
var apiService = builder.AddProject<Projects.AspireSample_ApiService>("apiservice");  
  
var ollama = builder.AddOllama(name: "ollama", port: 11434)  
    .WithOpenWebUI()  
    .WithDataVolume()  
    .PublishAsContainer()  
    .AddModel("llama3");  
  
builder.AddProject<Projects.AspireSample_Web>("webfrontend")  
    .WithExternalHttpEndpoints()  
    .WithReference(cache)  
    .WaitFor(cache)  
    .WithReference(apiService)  
    .WaitFor(apiService)  
    .WithReference(ollama)  
    .WaitFor(ollama);      
  
builder.Build().Run();
```


Official doc [here](https://learn.microsoft.com/en-us/dotnet/aspire/community-toolkit/ollama?tabs=dotnet-cli%2Cdocker)

Add this to your wwwroot folder. 
![file:/Users/red/Documents/AI/DotnetLocalAI/AspireSample/AspireSample.Web/wwwroot/chat.png](file:///Users/red/Documents/AI/DotnetLocalAI/AspireSample/AspireSample.Web/wwwroot/chat.png)


### Web/AIApiService.cs

```csharp
using System.Text;  
using System.Text.Json;  
  
public class AIApiService  
{  
    private readonly HttpClient _httpClient;  
  
    public AIApiService()  
    {  
        _httpClient = new HttpClient();  
    }  
  
    public async Task<string> PullModelAsync(string ollamaUrl, string modelName)  
    {  
        var requestUri = $"{ollamaUrl}/api/pull";  
        var jsonContent = $"{{ \"name\": \"{modelName}\" }}";  
        var content = new StringContent(jsonContent, Encoding.UTF8, "application/json");  
  
        var response = await _httpClient.PostAsync(requestUri, content);  
        response.EnsureSuccessStatusCode();  
  
        var responseString = await response.Content.ReadAsStringAsync();  
        // split the responseString into an array of string with a new element for each appearance of the string '{"status":'  
        var responseArray = responseString.Split("{\"status\":");  
  
        // remove the char '}' from the each element of the array  
        var responseHtml = "";  
        for (int i = 0; i < responseArray.Length; i++)  
        {  
            responseArray[i] = responseArray[i].Replace("}", "");  
            responseHtml += $"<p>{responseArray[i]}</p>";  
        }  
  
  
        return responseHtml;  
    }  
}
```

This code defines a class `AIApiService` that communicates with an external AI API. The primary focus of the class is to send a model request to a specific API endpoint and process the response into an HTML format.


This will allow us to make a post call to our local AI model. 
### Web/Program.cs
```csharp
using AspireSample.Web;  
using AspireSample.Web.Components;  
using Microsoft.Extensions.AI;  
  
var builder = WebApplication.CreateBuilder(args);  
  
builder.AddServiceDefaults();  
builder.AddRedisOutputCache("cache");  
  
builder.Services.AddRazorComponents()  
    .AddInteractiveServerComponents();  
  
builder.Services.AddHttpClient<WeatherApiClient>(client =>  
    {  
        client.BaseAddress = new("https+http://apiservice");  
    });  
  
builder.Services.AddLogging();  
builder.Logging.ClearProviders();  
builder.Logging.AddConsole();  
builder.Logging.AddDebug();  
builder.Logging.AddEventSourceLogger();  
builder.Logging.AddFilter("Microsoft", LogLevel.Information);  
builder.Logging.AddFilter("System", LogLevel.Information);  
builder.Logging.AddFilter("YourProject's Name", LogLevel.Debug);  
  
builder.WebHost.UseStaticWebAssets();  
  
builder.Services.AddSingleton<ILogger>(static serviceProvider =>  
{  
    var lf = serviceProvider.GetRequiredService<ILoggerFactory>();  
    return lf.CreateLogger(typeof(Program));  
});  
  
builder.Services.AddSingleton<IChatClient>(static serviceProvider =>  
{  
    var logger = serviceProvider.GetRequiredService<ILogger>();  
    var config = serviceProvider.GetRequiredService<IConfiguration>();  
    var AICnnString = config.GetConnectionString("ollama");  
    var LLM = config.GetConnectionString("llama3");  
    logger.LogInformation("AI connection string: {0}", AICnnString);  
    logger.LogInformation("LLM: {0}", LLM);  
  
    var chatClient = new OllamaChatClient(new Uri(AICnnString), LLM);  
  
    return chatClient;  
});  
  
builder.Services.AddSingleton<List<ChatMessage>>(static serviceProvider =>  
{  
    return new List<ChatMessage>()  
    { new ChatMessage(ChatRole.System, "You are a useful assistant that replies using short and precise sentences.")};  
});  
  
var app = builder.Build();  
  
if (!app.Environment.IsDevelopment())  
{  
    app.UseExceptionHandler("/Error", createScopeForErrors: true);  
    app.UseHsts();  
}  
  
app.UseHttpsRedirection();  
  
app.UseStaticFiles();  
app.UseAntiforgery();  
  
app.UseOutputCache();  
  
app.MapRazorComponents<App>()  
    .AddInteractiveServerRenderMode();  
  
app.MapDefaultEndpoints();  
  
app.Run();
```

Here we are setting up the dependencies. 

We will also create a class for processing the message

## MessageProcessor.cs

```csharp
using System.Text;  
using System.Text.Encodings.Web;  
using System.Text.RegularExpressions;  
using Microsoft.AspNetCore.Components;  
  
namespace AspireApp.WebApp.Chatbot;  
  
public static partial class MessageProcessor  
{  
    public static MarkupString AllowImages(string message)  
    {  
        // Processing markdown and deal with HTML encoding
        var result = new StringBuilder();  
        var prevEnd = 0;  
        message = message.Replace("&lt;", "<").Replace("&gt;", ">");  
  
        foreach (Match match in FindMarkdownImages().Matches(message))  
        {  
            var contentToHere = message.Substring(prevEnd, match.Index - prevEnd);  
            result.Append(HtmlEncoder.Default.Encode(contentToHere));  
            result.Append($"<img title=\"{(HtmlEncoder.Default.Encode(match.Groups[1].Value))}\" src=\"{(HtmlEncoder.Default.Encode(match.Groups[2].Value))}\" />");  
  
            prevEnd = match.Index + match.Length;  
        }  
        result.Append(HtmlEncoder.Default.Encode(message.Substring(prevEnd)));  
  
        return new MarkupString(result.ToString());  
    }  
  
    public static MarkupString ProcessMessageToHTML(string message)  
    {  
        return new MarkupString(message);  
    }  
  
    [GeneratedRegex(@"\!?\[([^\]]+)\]\s*\(([^\)]+)\)")]  
    private static partial Regex FindMarkdownImages();  
}
```


## ChatState.cs

```csharp
using Microsoft.Extensions.AI;  
using System.Security.Claims;  
  
namespace AspireApp.WebApp.Chatbot;  
  
public class ChatState  
{  
    private readonly ILogger _logger;  
    private readonly IChatClient _chatClient;  
    private List<ChatMessage> _chatMessages;  
  
    public List<ChatMessage> ChatMessages { get => _chatMessages; set => _chatMessages = value; }  
  
    public ChatState(ClaimsPrincipal user, IChatClient chatClient, List<ChatMessage> chatMessages, ILogger logger)  
    {  
        _logger = logger;  
        _chatClient = chatClient;  
        ChatMessages = chatMessages;  
    }  
  
    public async Task AddUserMessageAsync(string userText, Action onMessageAdded)  
    {  
        ChatMessages.Add(new ChatMessage(ChatRole.User, userText));  
        onMessageAdded();  
  
        try  
        {  
            _logger.LogInformation("Sending message to chat client.");  
            _logger.LogInformation($"user Text: {userText}");  
  
            var result = await _chatClient.CompleteAsync(ChatMessages);  
            ChatMessages.Add(new ChatMessage(ChatRole.Assistant, result.Message.Text));  
              
            _logger.LogInformation($"Assistant Response: {result.Message.Text}");  
        }  
        catch (Exception e)  
        {  
            if (_logger.IsEnabled(LogLevel.Error))  
            {  
                _logger.LogError(e, "Error getting chat completions.");  
            }  
  
            ChatMessages.Add(new ChatMessage(ChatRole.Assistant, $"We encountered an unexpected error.\n\n<p style=\"color: red\">{e}</p>"));  
        }  
        onMessageAdded();  
    }  
}
```

We will have a floating component to interact with the AI model. By adding a new component in the `MainLayout.razor` page under where we are rendering the body. So it will show up on every page we will call it ShowChatbotButton.

### ShowChatbotButton.razor

```csharp
@inject NavigationManager Nav  
  
<a class="show-chatbot" href="@Nav.GetUriWithQueryParameter("chat", true)" title="Show chatbot"></a>  
  
@if (ShowChat)  
{  
    <Chatbot />  
}  
  
@code {  
    [SupplyParameterFromQuery(Name = "chat")]  
    public bool ShowChat { get; set; }  
}
```

Then we we will make Chatbot.

## Chatbot.razor
```csharp
@rendermode @(new InteractiveServerRenderMode(prerender: false))  
@using Microsoft.AspNetCore.Components.Authorization  
@using AspireApp.WebApp.Chatbot  
@using Microsoft.Extensions.AI  
@inject IJSRuntime JS  
@inject NavigationManager Nav  
  
@inject AuthenticationStateProvider AuthenticationStateProvider  
@inject ILogger Logger  
@inject IConfiguration Configuration  
@inject IServiceProvider ServiceProvider  
  
<div class="floating-pane">  
    <a href="@Nav.GetUriWithQueryParameter("chat", (string?)null)" class="hide-chatbot" title="Close Chat"><span>✖</span></a>  
  
    <div class="chatbot-chat" @ref="chat">  
        @if (chatState is not null)  
        {  
            foreach (var message in chatState.ChatMessages.Where(m => m.Role == ChatRole.Assistant || m.Role == ChatRole.User))  
            {  
                if (!string.IsNullOrEmpty(message.Contents[0].ToString()))  
                {  
                    <p @key="@message" class="message message-@message.Role">@MessageProcessor.AllowImages(message.Contents[0].ToString()!)</p>                      
}  
            }  
        }  
        else if (missingConfiguration)  
        {  
            <p class="message message-assistant"><strong>The chatbot is missing required configuration.</strong> Please review your app settings.</p>  
        }  
  
        @if (thinking)  
        {  
            <p class="thinking">[@Configuration["Aspire:OllamaSharp:ollama:Models:0"]] is Thinking...</p>  
        }  
  
    </div>  
  
    <form class="chatbot-input" @onsubmit="SendMessageAsync">  
        <textarea placeholder="Start chatting..." @ref="@textbox" @bind="messageToSend"></textarea>  
        <button type="submit" title="Send" disabled="@(chatState is null)">Send</button>  
    </form>  
</div>  
  
@code {  
    bool missingConfiguration;  
    ChatState? chatState;  
    ElementReference textbox;  
    ElementReference chat;  
    string? messageToSend;  
    bool thinking;  
    IJSObjectReference? jsModule;  
  
    protected override async Task OnInitializedAsync()  
    {  
        IChatClient chatClient = ServiceProvider.GetService<IChatClient>();  
        List<ChatMessage> chatMessages = ServiceProvider.GetService<List<ChatMessage>>();  
        if (chatClient is not null)  
        {  
            AuthenticationState auth = await AuthenticationStateProvider.GetAuthenticationStateAsync();  
            chatState = new ChatState(auth.User, chatClient, chatMessages, Logger);  
        }  
        else  
        {  
            missingConfiguration = true;  
        }  
    }  
  
    private async Task SendMessageAsync()  
    {  
        var messageCopy = messageToSend?.Trim();  
        messageToSend = null;  
  
        if (chatState is not null && !string.IsNullOrEmpty(messageCopy))  
        {  
            thinking = true;  
            await chatState.AddUserMessageAsync(messageCopy, onMessageAdded: StateHasChanged);  
            thinking = false;  
        }  
    }  
  
    protected override async Task OnAfterRenderAsync(bool firstRender)  
    {  
        jsModule ??= await JS.InvokeAsync<IJSObjectReference>("import", "/Components/ChatBot/Chatbot.razor.js");  
        await jsModule.InvokeVoidAsync("scrollToEnd", chat);  
  
        if (firstRender)  
        {  
            await textbox.FocusAsync();  
            await jsModule.InvokeVoidAsync("submitOnEnter", textbox);  
        }  
    }  
}
```


the css and js for this as well.

## Chatbot.razor.js

```js
export function scrollToEnd(element) {  
    element.scrollTo({ top: element.scrollHeight, behavior: 'smooth' });  
}  
  
export function submitOnEnter(element) {  
    element.addEventListener('keydown', event => {  
        if (event.key === 'Enter') {  
            event.target.dispatchEvent(new Event('change'));  
            event.target.closest('form').dispatchEvent(new Event('submit'));  
        }  
    });  
}
```

## Chatbot.razor.css
```css
.floating-pane {  
    position: fixed;  
    padding-top: 1em;  
    width: 25rem;  
    height: 35rem;  
    right: 3rem;  
    bottom: 3rem;  
    border: 0.0625rem solid silver;  
    border-radius: 0.5rem;  
    background-color: white;  
    display: flex;  
    flex-direction: column;  
    font-weight: 400;  
    font-family: "Segoe UI", arial, helvetica;  
    animation: slide-in-from-right 0.3s ease-out;  
    z-index: 2;  
}  
  
@keyframes slide-in-from-right {  
    0% {  
        transform: translateX(30rem);  
    }  
    100% {  
        transform: translateX(0);  
    }  
}  
  
.hide-chatbot {  
    border: none;  
    background-color: #B4B4B8;  
    color: white;  
    position: absolute;  
    top: 0.25rem;  
    right: 0.18rem;  
    border-radius: 0.55rem;  
    width: 2rem;  
    height: 2rem;  
    z-index: 10;  
    text-decoration: none;  
    display: flex;  
    justify-content: center;  
    align-items: center;  
}  
  
.chatbot-input {  
    margin-top: auto;  
    display: flex;  
    position: relative;  
    border: 0.125rem solid #f4f0f4;  
    border-radius: 0.5rem;  
    padding: 0.5rem;  
    margin: 0.5rem 0.75rem;  
    gap: 0.3rem;  
    height: 3.5rem;  
    align-items: stretch;  
    flex-shrink: 0;  
}  
  
.chatbot-input textarea {  
    width: 100%;  
    background: none;  
    border: none;  
    outline: none;  
    resize: none;  
    font-weight: 400;  
    font-family: "Segoe UI", arial, helvetica;  
    font-size: 16px;  
}  
  
.chatbot-input button {  
    width: 6.25rem;  
    height: 3.125rem;  
    border-radius: 0.5rem;  
    border-width: 0;  
    cursor: pointer;  
    font-size: 0.875rem;  
    font-weight: 500;  
    padding: 0.6rem 0.8rem;  
    text-align: center;  
    margin-right: -7px;  
    margin-top: -7px;  
}  
  
.chatbot-chat {  
    overflow-y: scroll;  
    height: 100%;  
    padding: 0.5rem 0.75rem;  
    display: flex;  
    flex-direction: column;  
}  
  
.chatbot-chat .message {  
    padding: 0.5rem 1rem;  
    border-radius: 1.25rem;  
    max-width: 85%;  
    display: inline-block;  
    white-space: break-spaces;  
    overflow-x: clip;  
    margin-bottom: 0.75rem;  
    margin-top: 0.25rem;  
}  
  
.chatbot-chat .message-assistant {  
    background-color: #f4f0f4;  
    margin-right: auto;  
}  
  
.chatbot-chat .message-user {  
    background-color: #102c57;  
    margin-left: auto;  
    color: white;  
}  
  
.chatbot-chat .message-error {  
    background-color: #102c57;  
    margin-left: auto;  
    color: red;  
}  
  
.chatbot-chat ::deep img {  
    max-height: 10rem;  
}  
  
.thinking {  
    color: gray;  
    font-style: italic;  
    animation: fade-in-and-out 1s infinite;  
    padding: 0;  
    margin: 0;  
    padding-left: 0.6rem;  
    font-size: 90%;  
}  
  
@keyframes fade-in-and-out {  
    0% {  
        opacity: 0.2;  
    }  
  
    50% {  
        opacity: 0.9;  
    }  
  
    100% {  
        opacity: 0.2;  
    }  
}
```

If you want to run you need the commands below

```
dotnet dev-certs https --trust
```

```
dotnet run --project projectName/projectName.AppHost
```


That will get you to this point. Here we have set up what is going to download.

At this point you should see ![[Pasted image 20241218120120.png]]
This download may take a while depending on internet connection. 

---

## Conclusion

We are able to now test local workflows using local AI models. Because we are using Aspire with a few bicep scripts we can push this to Azure and flip to the real thing at any time. 