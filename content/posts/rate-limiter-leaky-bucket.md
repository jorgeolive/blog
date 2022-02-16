---
title: "Implementing a rate limiter in C# : II - Leaky bucket algorithm"
description: This time is all about semaphores!
htmlMetadata: Coding an Asp.net middleware to enqueue requests and FIFO dispatch them on a fixed rate.
image: 'semaphoreResized.jpg'
tags: algorithms; systems-design
postNumber: 2
date: 'February 12, 2022'
---

Okay, after the rather simple token bucket rate limiter (RL from now on) that we implemented on the [first entry of the series](https://www.jorge-olive.net/posts/rate-limiter-token-bucket) , we're ready to take it one step forward. Why so? Well, this first RL type has a weakness: while it ensures a max rate of hits, it doesn't care about their distribution in time. Putting it more clearly, On a 10 req/second spec'ed token bucket RL, there're no differences to receive 10 requests within a 10 ms window than having them spaced on 100ms intervals. It's clear that the former can compromise the upstream resource stability, so let's find out a mechanism to handle such situations.

# Leaky bucket algorithm

We can think of this algorithm as a **funnel on which the requests will be enqueued in a FIFO fashion and dispatched on a fixed rate. It will have a max capacity, and requests coming when full will be simply discarded.** There's not much else to it; it's very simple actually. But, don't let this simplicity missguide you; It ain't no whiteboard exercise; In fact, this is one of the most populars RL algorithms, used by a number of technologies, from reverse proxies - such as NGINX - to network hardware. One could argue that instead of rate limiting, it's more oriented to manage **congestion control** concerns. Fair point, but for the purpose of this series, we will consider it as the former.

# C# implementation

Let's analyze briefly what will be happening when an incoming request reaches the middleware:

- The component will check whether the bucket is full or not; 
- If not empty, it will be enqueued and released when its turn comes.
- If empty, it will be discarded.

Again, we will handle the logic within a middleware that will be run before any controller action. Each request thread will create a semaphore within the middleware that will be managed and released by the singleton RL meaning that the request will be able to continue once the RL decides. Also, the middleware component will be waiting to the semaphore to be released in order to progress further into the pipeline. But, what is a semaphore? Similar to what an OS Mutex is, a semaphore is an object that allows us to block threads reaching a segment of code. Any thread that reaches a "closed" semaphore will be blocked - async or sync depending on our needs - , and on a new available slot on it, a thread will be able to pass through it. It's important to note here that a semaphore is not a FIFO queue, so it could be the case that a thread is holded indefinitely waiting; that's why we need some additional type helping us preserve the FIFO nature of the design. ```ConcurrentCollection``` perhaps? 

```csharp
public class LeakyBucketRateLimiterMiddleware
    {
        private readonly RequestDelegate _next;

        public LeakyBucketRateLimiterMiddleware(RequestDelegate requestDelegate)
        {
            _next = requestDelegate;
        }

        public async Task InvokeAsync(HttpContext context, LeakyBucket bucket)
        {
            var semaphore = new SemaphoreSlim(0, 1);

            if (bucket.TryEnqueue(semaphore))
            {
                await semaphore.WaitAsync();
                await _next(context);
                //
            }
            else
            {
                context.Response.StatusCode = 429;
            }
        }
    }
```

Note the arguments passed to the ```SemaphoreSlim(0, 1)```: The semaphore will be closed initially - zero available initial requests - and it will be blocked once a single thread passes through it. 

The leaky bucket's concurrent collection will hold all the semaphores of the requests that "made it" as per the bucket size and are currently waiting. - *If you remember from the token bucket, ```ConcurrentCollection``` object is defined with a max size* -. A background thread that runs indefinitely will be pulling out the first semaphore on the queue and invoking its ```Release()``` method. This way, the middleware blocked thread will continue its execution.  

The last detail worth noting is the the actual semaphore release. The leaky bucket will be running an infinite background thread that, with the help of a ```Timer``` signaling the desired release rate through another semaphore, will take from the concurrent collection the first semaphore and release it:

```csharp
public class LeakyBucket
    {
        private System.Timers.Timer _timer;
        private BlockingCollection<SemaphoreSlim> _tokens;
        private SemaphoreSlim _timerSemaphore;

        public LeakyBucket(int bucketSize, decimal requestToDispatchBySecond)
        {
            _timerSemaphore = new SemaphoreSlim(0, 1);
            _tokens = new (bucketSize);
            InitTimer(requestToDispatchBySecond);

            // A backround task pulls the semaphores from the _tokens concurrent collection and releases them dictated by the Timer's managed semaphore "_timerSemaphore"
            Task.Run(async () =>
            {
                foreach(var requestSemaphore in _tokens.GetConsumingEnumerable())
                {
                    await _timerSemaphore.WaitAsync();
                    requestSemaphore.Release();
                }
            });
        }

        private void InitTimer(decimal requestToDispatchBySecond)
        {
            _timer = new System.Timers.Timer(Convert.ToDouble(1000/requestToDispatchBySecond));
            _timer.AutoReset = true;
            _timer.Elapsed += OnCountdown;
            _timer.Enabled = true;
        }

        private void OnCountdown(object? sender, ElapsedEventArgs e)
        {
            try
            {
                _timerSemaphore.Release();
            }
            catch (Exception)
            {
                ///Ignore exceptions thrown by the semaphore in case of race conditions
            }
        }

        public bool TryEnqueue(SemaphoreSlim semaphore)
        {
            return _tokens.TryAdd(semaphore);
        }
    }
```

And that's it. Hopefully the implementation was not too convoluted! 😅 

Feel free to take a look at the source code [here](https://github.com/jorgeolive/ratelimiter-leakybucket). Besides the sample code included in the snippets, there's a k6 script to run a test that demonstrates the behavior through the logs written by the sample Api. 

Until the next post!
