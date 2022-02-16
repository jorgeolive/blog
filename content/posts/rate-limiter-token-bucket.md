---
title: "Implementing a rate limiter in C# : I - Token bucket algorithm"
description: In this very first post, we will implement the simplest rate limiter posible in C#.
htmlMetadata: In this very first post, we will implement the simplest rate limiter posible in C# using the token bucket algorithm
image: 'bucket-169.jpeg'
tags: algorithms; systems-design
---

Rate limiters are no stranger in system design interviews. But, beyond the high level detail that is typically discussed on them, have you ever wondered how they are implemented ? For sure, they bring some interesting challenges - mostly concurrency related - worth to be looked at, so why not go ahead and naively implement some of their variants?

# What's a rate limiter, anyway?

Let's imagine a funnel on top of a bucket for a moment. No matter how much water you pour on it, only a part of the flow will reach the bucket, being the rest spilled away. Also, the speed on which the bucket will be filled, will depend on the funnel's end diameter. Translating this into systems context, the funnel would be the rate limiter and the system your bucket. 

So we can think of a rate limiter as **a layer that ensures that input that targets whatever is below it is throttled according to a specific criteria**. On paper, it's a very simple concept; but as we will see soon the devil is in the details. Before diving deeper, we can enumerate a few of it usages:

- With a rate limiter, you can ensure that your resource consumers won't go wild making an use of them above what is reasonable / expected. It's understood that such a situation can adversely affect your upstream services, hence, your other customers. Not good. 

- It can be used as well to help manage malicious DoS (Denial of Service) attacks. This can be done on different OSI layers, but in this post we will be talking about Application level ( OSI 7) ones. If you're using a rate limiter with this goal in mind, you'll need to consider whether placing it in-process within your application or on a completely separate component *a la* api gateway - reverse proxy.

Implementing a good rate limiter can be tricky; There're a variety of algorithms on how to control the flux of varying complexity, and the implementation might vary wildly depending on whether you're working in a distributed environment. That, and not to mention give it the flexibility to support all kinds of configurations. That's the reason why there are several consolidated commercial products that already implement them - eg: [Nginx](https://www.nginx.com/blog/rate-limiting-nginx/)-. Worth mentioning, OSS community has come up with really interesting propositions as well, such as dotnet's [AspNetCoreRateLimit](https://github.com/stefanprodan/AspNetCoreRateLimit).

In the next sections, we will take a look at the internals of a rate limiter visiting - and write - some of the algorithms that their core implementation is based on.

# Enter token bucket algorithm

The token bucket is probably the simplest of all the rate limiting algorithms, so it will be a good warm up to implement more complex stuff later on. It consists of a bucket holding a configured max number of tokens. Clients will "claim" and consume them in order to access the resource; obviously, when the token is consumed, it will be instantly removed from the bucket. On the meanwhile, another thread will be refilling this token bucket on a fixed frequency. Therefore, this can be reduced to "a given resource will be only hit n times per unit of time", being n the maximum number of tokens. Though it somewhat controls the traffic, it won't protect the resource from bursts of the bucket's capacity size.

# Naive C# implementation

We will implement the rate limiter as a middleware that will be called by every request. Depending on the policy - eg: rate limit by IP, user, etc. - or our intention (DoS protection mechanism), we might need to place it before or after other components such as authorization ones, but for now, we will keep it simple, limiting all the requests regardless of their origin.

The rate limiter is a **singleton thread-safe component** on which all the threads will pass through; Middleware instances are singleton too, so we inject our TokenBucket there. So, basically what we will do is, make all http requests request a token - which as we'll see, is nothing but an empty class - that will be extracted from the bucket. If there're no tokens available, the middleware will short circuit the request and the client will receive a 503 "*Service Not available*" http status code. If the rate limiter is based on client IP, probably a 429 *Too many requests* will be more explicit on what is going on.

```csharp
public class TokenBucketRateLimiterMiddleware
{
    private readonly RequestDelegate _next;

    public TokenBucketRateLimiterMiddleware(RequestDelegate requestDelegate)
    {
        _next = requestDelegate;
    }

    public async Task InvokeAsync(HttpContext context, TokenBucket bucket)
    {
        try
        {
            bucket.UseToken();
            await _next(context);
        }
        catch (NoTokensAvailableException)
        {
            context.Response.StatusCode = 503;
        }
    }
}
```
Let's dig now into the ```TokenBucket``` implementation. On one hand, it has a  ```ConcurrentCollection``` object holding instances of the dummy `Token` type. Actually, this is the core of the component, as it allows us to **concurrently consume items from its internal concurrent queue, whilst the producers concurrently put items into it.**

 Middleware's ```InvokeAsync``` method calls will request the tokens from there. For that, we will take advantage of the  ```TryTake``` method; which will return a true / false depending on whether an item was present on its internal concurrent queue. It's important to understand the differences here with the ```Take``` method, calling ```Take``` will return an object or it will block the caller until an item is present. We don't want that, as the request has to be ditched straight away because of the requirement. 

```csharp
public class TokenBucket
{
    private BlockingCollection<Token> _tokens;
    private int _maxTokens;

    public TokenBucket(int maxNumberOfTokens, int refillRateInMilliseconds)
    {
        _maxTokens = maxNumberOfTokens;
        _tokens = new BlockingCollection<Token>(maxNumberOfTokens);
    }

    public void UseToken()
    {
        if (!_tokens.TryTake(out Token _))
        {
            throw new NoTokensAvailableException();
        }
    }
}

public record Token;
```
<figcaption style="font-style: italic; font-size: 1.1rem;">Incomplete TokenBucket implementation: Managing concurrency</figcaption>


On the other hand, we need to take care of the periodical refill of the tokens. The ```System.Timers``` namespace already provides a ```Timer``` type that will really be handy for such purpose, as it allows to execute a delegate when the timer gets to zero on an infinite loop fashion:

```csharp
public class TokenBucket
{
    private BlockingCollection<Token> _tokens;
    private System.Timers.Timer _timer;
    private int _maxTokens;

    public TokenBucket(int maxNumberOfTokens, int refillRateInMilliseconds)
    {
        _maxTokens = maxNumberOfTokens;
        _timer = new System.Timers.Timer(refillRateInMilliseconds);
        _tokens = new BlockingCollection<Token>(maxNumberOfTokens);
        Init(maxNumberOfTokens);
    }

    private void Init(int maxNumberOfTokens)
    {
        foreach (var _ in Enumerable.Range(0, maxNumberOfTokens))
            _tokens.Add(new Token());

        _timer.AutoReset = true;
        _timer.Enabled = true;
        _timer.Elapsed += OnTimerElapsed;
    }

    private void OnTimerElapsed(object? sender, System.Timers.ElapsedEventArgs e)
    {
        foreach (var _ in Enumerable.Range(0, _maxTokens - _tokens.Count))
            _tokens.Add(new Token());
    }

    public void UseToken()
    {
        if (!_tokens.TryTake(out Token _))
        {
            throw new RateLimiterException();
        }
    }
}

public record Token;
```

When the timer gets to zero, we refill as many tokens as needed. Please note that there could be some race conditions here depending on how the thread scheduler gives priority to the timer thread vs the incoming Http requests and on whether the token refill is executed within the timer's thread execution window slice.

Almost there! at this point we only need to register this on the ```Program.cs``` and run some load tests:

```csharp
// Code ommited for brevity.
builder.Services.AddSingleton<TokenBucket>(new TokenBucket(maxNumberOfTokens: 2, refillRateInMilliseconds: 1000));

///..
```
With this configuration, only two requests per second should be hitting the controller. To check that, we will simply log in the WeatherForecast endpoint the current hour taking advantage of the new ```TimeOnly``` dotnet type:

```csharp
app.MapGet("/weatherforecast", () =>
{
    app.Logger.LogInformation("Request received at " + TimeOnly.FromDateTime(DateTime.UtcNow).ToLongTimeString());

///rest of endpoint code ommited for brevity 
```
Finally, running the k6 script we can see how out of all the requests, only aproximately 2 per second are actually reaching the endpoint:

 <nuxt-img src="token-bucket-loadtest.jpg" sizes="sm:320px md:650px lg:700px xl:1000px xxl:1500px"></nuxt-img>

So that's it for now. You can check the source code at [https://github.com/jorgeolive/token-bucket-rate-limiter](https://github.com/jorgeolive/token-bucket-rate-limiter)

Stay tuned for the next entry in the series!