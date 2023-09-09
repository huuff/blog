---
date: 2023-09-09
---

# The perfect size for REST interchanges
I've always been an advocate for small-sized REST messages. However, recently my view is starting to drift towards larger-grained messages (at least for responses). In this post I want to explore a few of benefits and drawbacks I'm considering regarding each choice.

## Smaller payloads are simpler to understand for humans
### Regarding requests
I've always found it quite disheartening when an HTTP service asks you to provide a thousand different parameters for any request. I still find that smaller requests are the way to go for a few reasons:

* They ease experimentation for beginning users: if your requests only take a few parameters, a user can start using them directly with a few `cURL` commands.
* This experimentation on its own directly produces an improvement on discoverability. Since users can get around your service much more easily, the need for reading huge API-specification documents diminishes and the barrier of entry lowers.
* Furthermore, they are great for `GET` requests. Since `GET` doesn't allow including entity bodies, all data must go into the query parameters. This gets unwieldy pretty quick (around 4 parameters already feels like too much). Using `GET` is great since it's the only safe and idempotent HTTP method that can return a resource representation, which makes it easily cacheable by clients. This is a long-standing issue of HTTP for me, which doesn't seem to be anywhere close to resolving unless `QUERY`[^1] takes off (and that draft's expired).

Obviously, this shouldn't drive us towards artificially shortening requests by keeping more state on the server (through cookies, sessions, maybe even through creating new types of resources). For cookies and sessions, we're violating the statelessness principle of REST. Experimentation gets harder because longer sequences of requests are required to do the same, and session information just becomes another parameter to pass to the service, and observability is hampered since requests can't be understood in isolation. They might help with the third issue (simpler `GET` URIs) but in my opinion this is actually a deficiency of REST that should be fixed instead of requiring you to work around it.

### Regarding responses
I think otherwise of responses. Obviously, smaller responses are also easier to understand for humans. But at what cost? If a resource naturally includes some big fat information, would you remove it just to make it easier to parse for a human? We mustn't forget that in the end, programs are the most likely users of our API, even if they are written by humans.

Trying to make your responses simplers to human is directly at tension with making them simpler for computers: you may split your resources to not include too much information, but that'll require your machine clients to go through longer transition chains to get what they want, increasing complexity and decreasing performance.


[^1]: [The HTTP QUERY method](https://www.ietf.org/archive/id/draft-ietf-httpbis-safe-method-w-body-02.html)
