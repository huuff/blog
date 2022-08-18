# An Overview of CAP Theorem's Criticism
The CAP theorem is one of the most well-known results in distributed data systems around the world, and it's been so heavily misused and misunderstood that it has been rightly deemed as "infamous". Amazingly, but as usual in our field, people found out dozens of different ways of being wrong about the CAP theorem, so I'm compiling here some of the most intelligent critique I've found around about it.

## CAP's definitions are too strict
* Consistency in CAP terms means *linearizability*, which is actually the highest possible level of consistency. This means that any data systems with weaker consistency levels such as causal, sequential or eventual consistency are **not** consistent in CAP terms. So this forgoes of most practical real-world data systems.
* Availability in CAP terms means that **every** non-failing node must reply with non-error responses. This, once again, forgoes many real systems like MongoDB, which, during a network partition, and to prevent ending in a split-brain situation, prevent the nodes in the nodes in the minority side of the partition from responding, even if they are non-failing.

As a consequence, most real data systems[^1] are not linearizable (for example, MongoDB allows stale-reads even with the highest consistency level[^2]), and they're not available either (as pointed out by the previous point). This means that using the strict, highly theoretical definitions in the CAP theorem for analyzing real systems means we must relax them to do so, and, at this point, we might just as well use something else.

### Proposed solutions
The *harvest and yield* model kind of relaxes the strict definitions of CAP ==TODO: Finish this section, look up the definitions in the *Database System Internals* book==

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

