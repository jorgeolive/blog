---
title: "Implementing a rate limiter : I - Token bucket algorithm"
description: In this post, we will implement the simplest rate limiter posible in C#.
custom_field: this is my custom field
image: 'bucket.jpeg'
tags: algorithms; systems-design
---

# What's a rate limiter anyway?

Imagine a funnel on top of a bucket. No matter how many water you throw at it, only a part of the flow will be reaching it, being the rest of it thrown away. Translating this into system context, we can think of a layer that ensures that input that targets whatever is below it is throotled according to a specific criteria. On paper, it's a very simple concept; but as we will see soon the devil is in the details. What are the main usages to it?

- With a rate limiter, you can ensure that your resource consumers won't go wild making an use of them above what is reasonable / expected. It's understood that such situation can affect adverserly your system, hence, your other customers. Not good. 

- It can be used as well to help managing DOSS (Denial of Service) attacks, perhaps as a last line of defense when the lower layers have not been able to repel it. If you're using a rate limiter with this objective in mind, you'll need to consider whether placing it in-process with your application or on a complete separate component *ala* api gateway. 

Implementing a good rate limiter can be tricky; There're a variety of algorithms on how to control the flux of varying complexity, and the implementation might vary wildly depending on whether you're working in a distributed environment; that, and not to mention give it the flexibility to support all kind of configurations. That's the reason why there're several consolidated commercial products - insert ejemplos aqui - , though the OSS community has come up with really interesting propositions - insert C# rate limiter here -. 


# Enter token bucket algorithm

The token bucket is probably the simplest of all the rate limiting algorithms. It consists of a bucket holding a configured max number of tokens. Clients will "claim" and consume them order to access the resource; a the token is consumed, it is removed from the bucket. On the meanwhile, another thread will be refilling this token bucket by a given frequency. Therefore, this can be reduced to "a given resource will be only hit n times per unit of time", being n the maximum number of tokens. Though it somewhat controls the traffic, it won't protect the resource from bursts of the bucket's capacity size.

DIBUJO

- NO CONTROLA FLOWS MAXIMOS 
Let's implement this in C# over the almighty WeatherForecast sample WebApi. 

# C# implementation

We will implement the rate limiter as a middleware that will be called by every request. Depending on the policy - eg: rate limit by IP, user, etc. - or our intention (DoSS protection mechanism), we might need to place it before or after other components like authorization, but for now, we will keep it simple. 

The rate limiter is a singleton thread-safe component on which all the threads will pass through; Middleware instances are singleton too, so we will inject the dependency over the Invoke method:

REVISAR SI EL MIDDLEWARE ES SINGLETON Y LO PODEMOS EMBEBER AHI



 CONFIRM THIS PART

