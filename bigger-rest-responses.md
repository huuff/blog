---
date: 2023-09-09
---

# The perfect size for REST interchanges
The size of a REST interchange is obviously a byproduct of its needs. Our REST messages can only be as lean as the content we need to add, and no bigger than the content we can provide. It, however, has definitely the power to drive our design depending on the degree of the performance and simplicity we target for our APIs, and even the client use-cases we want to support.

I've always been a supporter of smaller-sized REST messages to decrease bandwidth usage, lower the learning curve and even for enforcing data consistency (I'll expand on this later). As of lately, however, my views are starting to shift towards the fatter end of the spectrum. In this post I want to explore all considerations I've entertained about the issue.

## Smaller payloads are simpler to understand for humans
First, I'll expose my usual train of thought for supporting small interchanges: small sizes usually correlate to simplicity.

### Regarding requests
I've always found it quite disheartening when an HTTP service asks you to provide a thousand different parameters for any request. I still find that smaller requests are the way to go for a few reasons:

* They ease experimentation for beginning users: if your requests only take a few parameters, a user can start using them directly with a few `cURL` commands.
* This experimentation on its own directly produces an improvement on discoverability. Since users can get around your service much more easily, the need for reading huge API-specification documents diminishes and the barrier of entry lowers.
* Furthermore, they are great for `GET` requests. Since `GET` doesn't allow including entity bodies, all data must go into the query parameters. This gets unwieldy pretty quickly (around 4 parameters already feels like too much). Using `GET` is great since it's the only safe and idempotent HTTP method that can return a resource representation, which makes it easily cacheable by clients. This is a long-standing issue of HTTP for me, which doesn't seem to be anywhere close to resolving unless `QUERY`[^1] takes off (and that draft's expired).

Obviously, this shouldn't drive us towards artificially shortening requests by keeping more state on the server. I've seen two ways of doing this in the past:

* Server-side sessions (maybe through cookies). These violate the statelessness principle of REST. Requests no longer are self-contained, and depend on the correct set of state transitions to be executed and understood, which makes them more complex and hampers observability on the server side. Keeping application state on the server also makes horizontal scalability harder. Even Fielding's dissertation speaks against their usage[^2].
* Introduction of new resources. This might align more with a resource-oriented architecture, but it still feels artificial if it's done merely for the purpose of simplifying requests. In the end, this just consists of transforming application state into resource state[^*1] for the sake of it. More resources mean more requests, which interact badly with the weak consistency of the web. This still implies longer chains of state transitions to achieve the same logical action, which still hampers observability since the full chain must be recorded and understood for debugging.

Both of these methods might still help with the third mentioned issue: simpler URIs for `GET` requests. However, the fact that `GET` is needed for appropriate caching is a deficiency of the HTTP protocol, so they might be considered more of a workaround than an actual solution.

### Regarding responses
I think otherwise of responses. Obviously, smaller responses are also easier to understand for humans. But at what cost? If a resource naturally includes some big fat information, would you remove it just to make it easier to parse for a human? We mustn't forget that in the end, programs are the most likely users of our API, even if they are written by humans.

Trying to make your responses simplers to human is directly at tension with making them simpler for computers: you may split your resources to not include too much information, but that'll require your machine clients to go through longer transition chains to get what they want, increasing complexity and decreasing performance.

[^*1]: The separation of application state and resource state is one of the main tenets of REST architecture. Application state refers to the current state of the client (the application) in a process through the service state (the resource state) where the server only suggest next possible states through links (hypermedia). This enables separation of concerns between client and server and also enables better server scalability.

[^1]: [The HTTP QUERY method](https://www.ietf.org/archive/id/draft-ietf-httpbis-safe-method-w-body-02.html)
[^2]: [Architectural Styles and the Design of Network-based Software Architectures, Chapter 6: Experience and Evaluation](https://www.ics.uci.edu/~fielding/pubs/dissertation/evaluation.htm)
