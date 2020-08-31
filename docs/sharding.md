---
title:  "Sharding in IOTA: A review"
description: IOTA has not produced a design so far. Below are my notes & thoughts on Hans' design, on why I believe it won't work, and on why IOTA should be prioritizing a solid sharding design over a Coordicide design & implementation.
---
# Sharding

TL;DR: Sharding is necessary for scaling a permissionless DLT. IOTA has not
produced a design so far. Below are my notes & thoughts on Hans' design, on why
I believe it won't work, and on why IOTA should be prioritizing a solid
sharding design over a Coordicide design & implementation.

Edit 2020-08-29: *Hans has said that he is working on an extended, more formal
write-up of his design. The below represents a summary of what is publicly
known as of 2020-08-27, to the best of my knowledge/skills. Will update this article when that lands.*

# Dynamic/Fluid Sharding (Hans' design)

- Hans has a 3-part blogseries, of which he finished 2 parts
- This is on his personal blog, and was - at the time of writing - not reviewed in any form by any other IF member
- I have created some additional documentation, most of which has been confirmed by Hans (though this was a while back):

![/sharding.png](/sharding.png)

I should note that there are hints that the IF is aware about some of the
problems (there is a reference to a discussion
[here](https://iota.cafe/t/data-sharding/1188), but the [discussion
itself](https://iota.cafe/t/my-take-on-sharding/360/4) is unfortunately locked
down). I have not heard that any further sharding documentation is upcoming.

Hans did say that he is going to post a video series soon.  I fundamentally
disagree that this is a good approach here, as a discussion on this would benefit
greatly from precise statements, algorithms and formulas. With IOTA specifically,
there is often the problem that it is unclear which stage (chrysalis, coordicide,
sharding, future...) a given statement refers to.
While I appreciate videos as an additional illustration method for complicated
concepts, the medium is also not precise and prone to handwaving.

### Problems with this approach

[Note: this is to the best of my abilities. Again, there is no proper design,
so it's plenty guesswork]

[I'm also simplifying the shard selection to be non-fluid for now. The
arguments below are obviously equivalent (e.g. assume smallest-possible
fixed-sized shards with each actor occupying multiple shards → fluid sharding
with a (temporarily) static configuration)]

- Double-spend detection requires knowing the payload of both double-spending transactions. But only hashes are propagated.
    - The assumption is that, in order to double-spend, you'd need to *book two outgoing transaction in the origin shard,* one for each destination shard. But only one of these will be accepted in the merkle tree of the origin shard.
    - There are multiple problems here, but I'm just going to focus on the most obvious one: The security of this scheme falls apart as soon as the attacker has a 51% majority in a single "shard" - they can just create a fake merkle tree to propagate. Because the destination shard doesn't know the payloads (they can't, that's the point of sharding), they can't discern a random hash from an actual transaction hash - therefore, they can't detect double spends.
    - Example: If you have a 100 shards, the attacker only needs 0.5% of the nodes (or mana) for a double-spend.
- Cross-shard transaction overhead is massive, and latency is high.
    - Each transaction needs to be "handed over" from each shard to the next one. Because you need consensus about the transaction on both the "outgoing" shard marker and the "incoming shard marker", the furthest each transaction can go in each step is the observation radius of a 51% quorum per shard. In practice, every node in every shard "along the way" will need to handle the transaction.
    - A transaction between two random shards will (on average) have 1/4th of the ring between them, so 1/4th of all nodes globally need to handle the transaction. For example, if you have 1000 shards, the transaction needs to do 250 steps to get to the destination.
    - From a (CPU, Network)-utilization perspective, this means that the maximum
      amount of global cross-shard TPS is only 4x of the TPS that a single shard
      could achieve.
    - It is hard to make proper claims on latency right now without knowing the
      coordicide performance characteristics. Assuming 5 second per step
      (which, IMHO is rather optimistic - the confirmation time in Chrysalis 1 is
      ~10 sec), we get an average transmission latency of (250 shards \* 5 seconds) = 20 minutes.

Now i'm not saying that there aren't approaches to deal with this - like adding validators, adding randomized shard selection etc., but these are core elements of the sharding design. *These are, in my opinion and experience, not add-ons that you can slap on later*. 

# General thoughts

- Sharded systems are either sharded from the start, or shard poorly. You don't "add sharding later on".
    - Examples:
    [MySQL](https://stackoverflow.com/questions/5541421/mysql-sharding-approaches) ("The best approach for sharding MySQL tables to not do it unless it is totally unavoidable to do it"), Ethereum (how long have they tried now?).
    - Databases that shard well, like DynamoDB or Apache Cassandra, were
      designed from scratch with sharding in mind. DLTs with more promising
      sharding approaches, such as Near (Nightshade), Radix (Cerberus) or Polkadot,
      **start** their whitepaper with a description of how they are going to shard:
      It's the most important design aspect, *everything else follows from that*.

- The Tangle does not "naturally shard", as it is occasionally claimed. If that
  was the case, the IF could have just simply dumped one Coordinator in each
  shard and would see ~linear TPS gains *right now*.
  
  Logically, a token like IOTA forms a single partition over which consensus needs to be achieved (otherwise, you can double-spend in a different shard, see CfB's "Economic Clustering" proposal which does exactly that).

# But....

## "So what? It's not finished then / It's called innovation"

The problem with this argument is that people are already investing under the assumption that sharding will exist:

- For proper industry adoption, and to run smart contracts with multiple input/output transaction per invocation, you certainly require more than a few hundred TPS.
    - (I will write a separate article about this at some point, but Coordicide turns the coordinator into a replicated statemachine - the TPS will be significantly lower than pre-coordicide. Additionally, White Flag will have to be removed again. While the new Tx layout should bring some gains I would not expect TPS above 50-100 sustained post-coordicide).
- As I've pointed out above, it is very hard (or even impossible) to add proper sharding to a system that is not designed for it, and IOTA is not designed for it.
- So whatever IOTA is building right now is *for nothing,* unless a proper long-term scaling plan exists: There's a good chance that the IF will have to throw everything out again and start with a design that actually supports sharding. Making all the innovation right now *wasted time & wasted money*.

## But we could also do other scaling solutions, like Layer 2

Maybe? I'm not familiar enough with layer 2 solutions (lightning network et al). Hans did propose a 'private subtangle' concept in the past, but I am not actually sure whether this is something that is being given any serious thought.
Also, a private subtangle (or, most L2 scaling solutions really) look a lot like a
cloud-based solutions - you lose the permissionless-aspect, you lose visibility
into transactions etc.. And a cloud based solution is of course (and
especially at high TPS) a fraction of the opex cost.

