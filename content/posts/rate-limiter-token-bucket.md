---
title: "Implementing a rate limiter in C# : I - Token bucket algorithm"
description: In this post, we will implement the simplest rate limiter posible in C#.
custom_field: this is my custom field
image: 'bucket.jpeg'
tags: algorithms; systems-design
---

# What's a rate limiter anyway?

Imagine a funnel on top of a bucket. No matter how many water you throw at it, only a part of the flow will be reaching the bucket, being the rest thrown away. Translating this into system context, we can think of a layer that ensures that input that targets whatever is below it is throotled according to a specific criteria. On paper, it's a very simple concept; but as we will see soon the devil is in the details. What are the main usages to it?

- With a rate limiter, you can ensure that your resource consumers won't go wild making an use of them above what is reasonable / expected. It's understood that such situation can affect adverserly your upstream services, hence, your other customers. Not good. 

- It can be used as well to help managing DOSS (Denial of Service) attacks, perhaps as a last line of defense when the lower OSI layers protection mechanisms have not been able to repel it. If you're using a rate limiter with this objective in mind, you'll need to consider whether placing it in-process with your application or on a complete separate component *ala* api gateway - reverse proxy.

Implementing a good rate limiter can be tricky; There're a variety of algorithms on how to control the flux of varying complexity, and the implementation might vary wildly depending on whether you're working in a distributed environment; that, and not to mention give it the flexibility to support all kind of configurations. That's the reason why there're several consolidated commercial products that already implements them - eg: Nginx -. Worth mentioning, OSS community has come up with really interesting propositions as well- insert C# rate limiter here -. 


# Enter token bucket algorithm

The token bucket is probably the simplest of all the rate limiting algorithms, so it will be a good warm up to implement more complex stuff later on. It consists of a bucket holding a configured max number of tokens. Clients will "claim" and consume them order to access the resource; obviously, when the token is consumed, it is instantly removed from the bucket. On the meanwhile, another thread will be refilling this token bucket by a given frequency. Therefore, this can be reduced to "a given resource will be only hit n times per unit of time", being n the maximum number of tokens. Though it somewhat controls the traffic, it won't protect the resource from bursts of the bucket's capacity size.

Let's implement this in C# over the almighty WeatherForecast sample WebApi. 

# C# implementation

We will implement the rate limiter as a middleware that will be called by every request. Depending on the policy - eg: rate limit by IP, user, etc. - or our intention (DoSS protection mechanism), we might need to place it before or after other components such as authorization ones, but for now, we will keep it simple. 

The rate limiter is a singleton thread-safe component on which all the threads will pass through; Middleware instances are singleton too, so we inject our TokenBucket there. So, basically what we will do is, make al Http requests request a token - which as we'll see, is nothing but a empty class - that will be extracted from the bucket. If there're no tokens available, the middleware will shortcircuit the request and client will receive a 503 "Service Not available" http status code. If the rate limiter is based on client IP, probably a 429 "Too many requests" will be more explicit on what is going on.

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
        catch ( NoTokensAvailableException ex)
        {
            context.Response.StatusCode = 503;
        }
    }
}
```
Let's dig now into the TokenBucket implementation. On one hand, it has a  <span style="color: red">```ConcurrentCollection```</span> object holding instances of the dummy `Token` type. Actually, this is the core of the implementation, as it allows us to concurrently consume items from its internal concurrent queue, while the producers concurrently put items into it.

 Middleware's <span style="color: red; font-weight: bold;">```InvokeAsync```</span> method calls will request the tokens from there. For that, we will take advantage of the  <span style="color: red">```TryTake```</span> method; which will return a true / false depending on whether an item was present on its internal concurrent queue. It's important to understand the differences here with the <span style="color: red">```Take```</span> method, calling <span style="color: red">```Take```</span> will return an object or it will block the caller - until an item is present. We don't want that, as the request has to be ditched straight away because of the requirement. 

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
*Incomplete TokenBucket implementation: Managing concurrency*


At this point, we need to take care of the periodical refill of the tokens. The <span style="color: red; font-weight: bold;">```System.Timers```</span> namespace already provides a <span style="color: red; font-weight: bold;">```Timer```</span> type that will really be handy for such purpose, as it allows to execute a delegate when the timer gets to zero on an infinite loop fashion:

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

When the timer gets to zero, we refill as many tokens as needed. Please note that there could be some race conditions here depending on how the scheduler gives priority to the Timer thread vs the incoming Http requests. 

Finally, we only need to register this on the <span style="color: red; font-weight: bold;">```Program```</span> and run some tests:

```csharp
// Code ommited for brevity.
builder.Services.AddSingleton<TokenBucket>(new TokenBucket(maxNumberOfTokens: 10, refillRateInMilliseconds: 1000));

///..

app.UseMiddleware<TokenBucketRateLimiterMiddleware>()
```
