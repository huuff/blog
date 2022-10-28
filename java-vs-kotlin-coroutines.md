# Coroutines
Coroutines allow a natural way of describing asynchronous operations as if they were synchronous.

## Coroutines survived loom, unlike reactive
I always find the emergence of Project Loom as a huge proof of Kotlin being future-proof. 

Traditionally the solution to provide high-performance asynchronous programming in the Java world has been reactive programming ==TODO: wrong term?==, but this has several drawbacks:
* It forces the programmer to follow a very unorthodox style (especially for non-functional languages, like Java), that also spreads like wildfire through the codebase and must be fully embraced to take advantage of reactive's performance improvements.
* It wildly obscures the stack trace of exceptions and is much harder to debug than synchronous code.
* It also forces the developer to sprinkle its types around the code, obscuring or removing some assurances from it. For example, a `Mono<T>` might contain a `T` or not. This might not be such a big deal for Java developers since they're already used to any type being nullable, but also consider that there's no way to take advantage of JSR-305 with reactive types.

Kotlin's coroutines instead, seamlessly adapted and provided transformations from reactive types to coroutines. Then Project Loom and fibers came around. Now **even Brian Goetz himself claims that fibers will kill reactive** ==TODO: Citation needed==. This means that all code written under a reactive style no longer has any purpose, and is needlesly obscured. Coroutines, on the other hand, were never very different to normal, non-reactive code. But even then, there's still benefits to coroutines besides performance, and they'll still be around ==TODO: citation needed==
