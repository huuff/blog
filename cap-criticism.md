# An Overview of CAP Theorem's Criticism
The CAP theorem is one of the most well-known results in distributed data systems around the world, and it's been so heavily misused and misunderstood that it has been rightly deemed as "infamous". Amazingly, but as usual in our field, people found out dozens of different ways of being wrong about the CAP theorem, so I'm compiling here some of the most intelligent critique I've found around about it.

## History
Our first stop in reviewing the actual definitions of CAP and how it stands against its criticism, should be finding out an authoritative source for it. A timeline of its development is helpful here:

1. 1999: First published in the *harvest* and *yield* paper[^1], for which it's not a centerpiece, not very formally specified and present almost like a quick rule-of-thumb. 
2. 2000: This was later presented in a well-known keynote[^2] which preserved its original spirit rather intact.
3. 2002: CAP gets formalized, and proved, making it a theorem, by Gilbert and Lynch[^3]
4. 2012: Eric A. Brewer, original author, comments on the conjecture himself[^4], where he contravenes some parts of its original statement.[^5]


## Real systems are neither CAP-consistent nor CAP-available
This comes down to the strict definitions of consistency and availability used to prove the CAP's theorem[^3]

* Consistency means *linearizability*, which is actually the highest possible consistency level. In a linearizable distributed systems, a read always returns the latest possible write for that value, and operations are ordered according to the real wall-clock time at which they were executed. This requires expensive coordination among nodes.
* Availability means that **every** non-failing node must return a valid response during a partition.

However, in real systems:

* Linearizability is forfeited in favor of some weaker consistency level that imposes less coordination overhead and leads to lower latencies. For example, *eventually consistent* systems do not always return the latest write, but they do it *eventually* given enough time. ==TODO: Note on MongoDB stale reads, use all of Jepsen analyses==
* Non-failing nodes are disallowed to respond if they are known to be unable to preserve consistency during a network partition. For example, in *quorum*-based systems, only the nodes in the majority side of the partition are allowed to respond, while non-failing nodes in the minority side are disallowed from doing so, as they might be lagging behind.

* Consistency means *linearizability*, which is actually the highest possible consistency level. Most real-world systems forfeit linearizability in favor of some weaker consistency level to decrease latency. This means that any *eventually consistent* system, is, by definition, not CAP-consistent.
* Availability means that every non-failing node must return a valid response. Usually, real-world systems work on a *quorum*-based mode during a partition, meaning that only nodes in the majority side of the partition will continue responding. This means they are not CAP-available, since non-failing nodes 

The take-away is that real-world systems are neither CAP-consistent nor CAP-available, and thus, neither CP or AP.

### Proposed solutions
CAP was originally stated along with the *Harvest and Yield*[^1] model. In this model, instead of such precise, stringent requirements, more relaxed and realistic probability-based definitions are used. ==TODO: complete this==

## Criticism on criticism
See what happened there? Several iterations on the original statement makes it a hard-to-hit moving target. Most criticism relates specifically to one, but not all, of the installments of the conjecture. As usual, formalizing the original conjecture into a theorem made it's definitions stricter, partially incompatible with the original ones presented in 1999, and almost sucked out all of the fun of it. I think it's unfair to criticise the use of CAP in informal settings (i.e. in a marketing piece for a database system that claims itself to be CA) by using its formal definitions from the proof from 2002. Nevertheless, I consider all of this rumination on any given definitions both entertaining and enlightening and thus I'm just compiling it as-is here.

## References
[^1]: [Harvest, Yield, and Scalable Tolerant Systems](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.33.411&rep=rep1&type=pdf)
[^2]: [Towards Robust Distributed Systems](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf)
[^3]: [Brewer's Conjecture and the Feasibility of Consistent, Available, Partition-Tolerant Web Services](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.67.6951&rep=rep1&type=pdf)
[^4]: [CAP Twelve Years Later: How the "Rules" Have Changed](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)


#### OLD STUFF, MUST REWORK IT

## You can't just "choose two"
Probably the most famous oversimplification of the CAP theorem is "Consistency, Availability and Partition Tolerance, choose two". However, this presents a number of inconveniences:

* It makes it seem like there's a strict proportional tradeoff between consistency and availability, and that increasing one will inherently decrease the other. Meanwhile, the reality is much more nuanced than that.
* As pointed out in the previous point, most real data systems can't choose any other than partition tolerance according to CAP's definitions.
* You definitely can't choose consistency and availability[^4], leaving out partition tolerance. I'll elaborate:

The CAP theorem is defined in terms of what happens during a network partition, and thus, removing network partitions from the model just opts-out of the CAP theorem. The tradeof in CAP is the following: in the event of a network partition, a node separate from the rest can choose to either:
* Accept incoming requests, preserving availability, but sacrificing consistency, since any data it responds with might be stale.
* Refuse incoming requests, sacrificing availability, but maybe preserving consistency, since nodes in another partition might form a majority and still be able to serve the latest writes.

As you can see, this is a binary choice and there is no way out of it. You might think that maybe opting out of partition tolerance works but:
* Partitions can definitely happen in any case. Even when your system can only reside on a single node (no replication and no sharding), a client might be partitioned from the served.
* If you claim that your data system is "not partition tolerance", what does this mean? Likely, that in the even of a partition (even in the single-node case), your system is unreachable and no longer serving any requests. But that's just sacrificing availability for consistency[^5]!

See? There's no way out of it

## It only models one kind of fault
==TODO: Write this section, make a note on PACELC as a solution. I've found some good texts on this, but I don't remember where==

[^1]: [Please stop calling databases CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)
[^2]: [Jepsen: MongoDB stale reads](https://aphyr.com/posts/322-call-me-maybe-mongodb-stale-reads)
[^3]: [Harvest, Yield, and Scalable Tolerant Systems](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.33.411&rep=rep1&type=pdf)
[^4]: [You Can't Sacrifice Partition Tolerance](https://codahale.com/you-cant-sacrifice-partition-tolerance/)
[^5]: [Problems with CAP, and Yahoo's little known NoSQL system](https://dbmsmusings.blogspot.com/2010/04/problems-with-cap-and-yahoos-little.html)

