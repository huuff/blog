# An Overview of CAP Theorem's Criticism
The CAP theorem is one of the most well-known results in distributed data systems around the world, and it's been so heavily misused and misunderstood that it has been rightly deemed as "infamous". Amazingly, but as usual in our field, people found out dozens of different ways of being wrong about the CAP theorem, so I'm compiling here some of the most intelligent critique I've found around about it.

## History
Our first stop in reviewing the actual definitions of CAP and how it stands against its criticism, should be finding out an authoritative source for it. A timeline of its development is helpful here:

1. 1999: First published in the *harvest* and *yield* paper[^1], for which it's not a centerpiece, not very formally specified and present almost like a quick rule-of-thumb. 
2. 2000: This was later presented in a well-known keynote[^2] which preserved its original spirit rather intact.
3. 2002: CAP gets formalized, and proved, making it a theorem, by Gilbert and Lynch[^3]
4. 2012: Eric A. Brewer, original author, comments on the conjecture himself[^4], where he contravenes some parts of its original statement.[^5]

## The proof, distilled
First, CAP requires three definitions[^3]:

* *Consistency*: Means *linearizability*, that is, all operations must appear as if they completed in a single point in time, and a total order can be established among them that is consistent with the real wall-clock time at which they were run.
* *Availability*: Means that **all** non failing nodes mut return a valid response.
* *Partition Tolerance*: Means that the network is allowed to lose arbitrarily many packets.

The crux of it: during a partition, you have to choose either consistency or availability, but not both. This is very clearly illustrated by a simple example: consider a single non-failing node partitioned from the rest of its cluster, and it receives a write request. Then, it has only two choices:

* Accept the request, even though it knows that other nodes it cannot reach won't be able to read this write until after the partition heals. Therefore, it forfeits *consistency* and chooses *availability*.
* Reject the request, so that any read can only be run on nodes that can definitelly return the latest agreed-upon write. Therefore, it forfeits *availability* and chooses *consistency*.

## Real systems are neither CAP-consistent nor CAP-available
This comes down to the strict definitions of consistency and availability used to prove the CAP's theorem[^3]

* Consistency means *linearizability*, which is actually the highest possible consistency level. But in there exist many weaker consistency levels that might be more practical in real systems.
* Availability means that **every** non-failing node must return a valid response. This is, however, not the metric that most vendors use to define *high availability* or SLAs.

Therefore, in real-world systems:

* Linearizability is forfeited in favor of some weaker consistency level that imposes less coordination overhead and leads to lower latencies. For example, *eventually consistent* systems do not always return the latest write, but they do it *eventually* given enough time. ==TODO: Note on MongoDB stale reads, use all of Jepsen analyses==
* Non-failing nodes are disallowed to respond if they are known to be unable to preserve consistency during a network partition. For example, in *quorum*-based systems, only the nodes in the majority side of the partition are allowed to respond, while non-failing nodes in the minority side are disallowed from doing so, as they might be lagging behind. However, this has no impact on the SLA of the application as it continues serving all requests.

The take-away is that real-world systems are neither CAP-consistent nor CAP-available, and thus, neither CP or AP.

### Proposed solutions
CAP was originally stated along with the *Harvest and Yield*[^1] model. In this model, instead of such precise, stringent requirements, more relaxed and realistic probability-based definitions are used:

* *Yield* is the probability of completing a request.
* *Harvest* is the fraction of data reflected in the response, i.e. the completeness of the answer to the query.

As exemplified in the paper, this removes the strict choice between consistency and availability: some systems might decrease *harvest* to keep a high *yield* during a fault, and viceversa. This is for example seen in partitioned (i.e. sharded) systems, where the failure of a shard does not prevent a request for completing, instead, the response just lacks the data in that shard (trading *harvest* for *yield*)

## Tradeoff confusion
CAP seems to make people think that choosing between consistency and availability is a binary choice, and that increasing one inherently decreases the other. As we saw in the previous point, the CAP theorem's proof concerns itself with a very specific situation, and almost all others fall out of its domain, so the tradeoff in real-world systems is not so black and white. But also, there are some subtler points:

* AP systems are much harder to pull off than CP systems. This is immediately obvious by realizing that the trivial system that ignores all requests is considered CP in the proof[^3]. But also considering that allowing *all nodes* to be available during a partition requires us to implement some conflict resolution mechanism for when the partition heals[^4], a field with a lot of complexity on its own. Preserving consistency during a partition in an already consistent system is not so hard: we can just failover to the majority side of the partition.

==TODO: finish this==

### Proposed solutions
==TODO: Mention PACELC and Harvest and Yield==

* Nicolas Liochon proves in a blog post[^10] that choosing between C and A (in CAP terms) has some value when considering real-time (i.e. time-bound) constraints.

## Can you choose CA?
In the original formulation of CAP, the famous adage was *Consistency, Availability, and Partition Tolerance. Choose two*. Well then, can you choose consistency and availability (CA), having a perfectly linearizable and available system, as long as there are no partitions?

There's a lot of discussion on this:

* If you take Eric Brewer as an authoritative source on this, in the original formulation[^1] and the subsequent keynote[^2] CA given as a possibility, but he later[^3] rejected the idea twelve years later[^4].
* Partitions happen whether your system handles them or not[^6]. In the extreme case, even a single-node system can be partitioned from a client running in the same computer (e.g. the socket is closed).
* The theorem's proof specifically concerns itself with a binary choice during a network partition: either a partitioned node accepts a message (choose availability) or rejects it (choose consistency). There is no third option.[^7]
* Considering that the proof accepts the trivial system that rejects all messages a CP system[^3], this means that even a single-node system that rejects all requests is a CP system, and thus forfeits availability.

### Proposed solutions
Considering CA as a different type of choice from CP or AP[^8][^9]. Specifically considering it as opting out of CAP and considering a CA system as one whose behavior is undefined during a partition. This is a weak solution for me, since partitions will still happen[^6], and knowing the system is unable to decide what'll happen on them is not at all reassuring, and also considering that this might be missleading, since the system might not be consistent nor available during a partition, as it's undefined. This, however, is a pragmatic definition since it's likely what most systems that have been historically claimed to be CA meant.

## It only models a very specific failure scenario
More specifically, there are many real-world failure scenarios not covered by it,

* The CAP theorem is not concerned with failed nodes[^5]

### Proposed solutions
==TODO: Take a note on PACELC as a solution to model the tradeoffs during normal operation==

## Criticism on criticism
See what happened there? Several iterations on the original statement makes it a hard-to-hit moving target. Most criticism relates specifically to one, but not all, of the installments of the conjecture. As usual, formalizing the original conjecture into a theorem made it's definitions stricter, partially incompatible with the original ones presented in 1999, and almost sucked out all of the fun of it. I think it's unfair to criticise the use of CAP in informal settings (i.e. in a marketing piece for a database system that claims itself to be CA) by using its formal definitions from the proof from 2002. Nevertheless, I consider all of this rumination on any given definitions both entertaining and enlightening and thus I'm just compiling it as-is here.

## References
[^1]: [Harvest, Yield, and Scalable Tolerant Systems](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.33.411&rep=rep1&type=pdf)
[^2]: [Towards Robust Distributed Systems](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf)
[^3]: [Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.67.6951&rep=rep1&type=pdf)
[^4]: [CAP Twelve Years Later: How the "Rules" Have Changed](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)
[^5]: [Don't use the CAP theorem for node failures](http://blog.thislongrun.com/2015/03/dead-nodes-dont-bite.html)
[^6]: [The network is reliable](https://aphyr.com/posts/288-the-network-is-reliable)
[^7]: [You Can't Sacrifice Partition Tolerance](https://codahale.com/you-cant-sacrifice-partition-tolerance/)
[^8]: [The unclear CP vs. CA case in CAP](http://blog.thislongrun.com/2015/04/the-unclear-cp-vs-ca-case-in-cap.html)
[^9]: [You Do It Too: Forfeiting Network Partition Tolerance in Distributed Systems](http://blog.thislongrun.com/2015/07/Forfeit-Partition-Tolerance-Distributed-System-CAP-Theorem.html)
[^10]: [If CAP were real-time: adding timing requirements to the definition of availability](http://blog.thislongrun.com/2015/06/real-time-CAP-theorem.html)
