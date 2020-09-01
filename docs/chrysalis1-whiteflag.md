---
title:  "White Flag is a Mistake"
description: White Flag gave IOTA a nice TPS boost. But it's temporary, increases dependence on Coordinator, and is bad for research, adoption & Coordicide development.
date: 2020-09-01
---

# White Flag is a Mistake

TL;DR: White Flag gave IOTA a nice TPS boost. But it's temporary, increases
dependence on Coordinator, and is bad for research, adoption & Coordicide
development.

# What is White Flag?

Let's quickly discuss what White Flag *does*, and how it relates to the IOTA/Tangle concept.

Simplified description:

- White Flag allows invalid (conflicting) transactions to remain in the DAG.
  The coordinator will approve transactions that are invalid (directly or
  referencing invalid ones). When a milestone is received, nodes take all the
  transactions since the last milestone, sort them (total order over
  DAG), and walk down the list of transactions. Transactions that conflict
  with a previous transaction are dropped.
- In other words: The Coordinator's milestones can now approve *anything*, and
  it is up to the nodes to figure out which transactions are valid and which
  ones aren't.
- This creates a 100% confirmation rate, because every transaction can now be
  considered valid. This also removes the need for reattach & promote, as there
  are no invalid branches anymore. It also simplifies GTTA, because every tip can
  be attached to, valid or not.
- But **White Flag is a temporary thing: It can't work with Coordicide**, so it
  will be removed again [("valid for the pre-coordicide era")](https://blog.iota.org/chrysalis-b9906ec9d2de).

At first glance, *this seems good*: It looks like this change reduces the
reliance on the coordinator - the coordinator doesn't actually *do anything*
anymore except just sign a random transaction. Seriously, any transaction that
indirectly references the previous milestone, valid or not, will do.

# Primary Drawbacks

Unfortunately, this also breaks virtually every assumption that people have
made about the Tangle, all the way down to "*one transaction approves two
others*". Some of the direct implications are pointed out in the
["Drawbacks" section of the RFC](https://github.com/thibault-martinez/protocol-rfcs/blob/rfc-white-flag/text/0005-white-flag/0005-white-flag.md#drawbacks):

> The ledger state is now only well-defined at milestones, meaning that we have to wait until each milestone is issued in order to confirm a spend;

In other words, a "transaction approving two others" doesn't mean anything any more, because transactions can now approve conflicting / invalid transactions)

> Everything that is seen is now part of the Tangle, including double-spend attempts, meaning that malicious data will now be saved as part of the consensus set of the Tangle;

In other words, just because something is "in the tangle" doesn't mean it's actually valid, which also means that... 

> To prove that a specific (non-milestone) transaction is valid, it is no longer sufficient to just provide the "path" to its confirming milestone, but instead all transactions in its past cone.

So you can't do merkle-proofs-of-inclusion with this.

# Implications for Researchers & Industry Adoption

- It makes the DAG-structure useless, and turns IOTA into a blockchain (a poor one, as each block includes invalid transactions).
    - Because the coordinator can approve invalid transactions, every other transaction can, too. I think this cannot be stressed enough: **Any transactions can approve invalid transactions, without any repercussions.**
    - This turns "one transaction approves two others" into "one transaction *references* two others". The references have no meaning, no value. Thus the whole concept of "securing the network with cumulative PoW (or VDF)" is meaningless.
    - This takes IOTA further away from where "it wants to be", it effectively makes "The Tangle" paper completely unrelated to IOTA.
- This makes it less suitable for **researchers** evaluating tangle properties. For example, they can't work with merkle proofs, they can't evaluate security properties ("PoW validates indirect aprovees"), can't evaluate performance (e.g. effects of reattach & promote)
- For **industry partners**, all of the above holds too, and creates additional uncertainties. Chrysalis makes it *harder* for them to evaluate how the network will perform post-coordicide. The same holds for open source developers & the ecosystem community, of course. Also, nobody in the industry seems to actually *require* the TPS increase right now:
    - Hornet already boosted the TPS quite a bit over IRI - but mainnet is running at 15 TPS at the time of writing, well below both the pre- and post-WF limits, all spam.
    - Private tangles exist as options for higher-tps testing.
    - Industry partners are not going to start seriously evaluating IOTA until Chrysalis Pt. 2 is done, because it changes almost everything anyway.

# White Flag makes Coordicide way harder

Coordicide will necessearily make the network slower, WF or not (because you're
turning a centralized process into a replicated one, it's slower by definition.
Also, added overhead from Mana, FPC, etc).

But removing White Flag will slow down Coordicide *a lot more*. If I remember
correctly, Hornet gained a ~5x speedup with White Flag. And this will need to
be undone. I would imagine a 5x slowdown to be a tough sell to the community.

The alternative would be for the developers to invest additional time and
effort in making White Flag work *somehow* on the post-coordicide network, thus
making Coordicide even more complex (and delayed) as it already is.

Either way, painful.


# Summary & Chrysalis Phase 2

TL;DR (2): White Flag cost months of engineering time to build, there is no
short-term requirements to do this, there are no long-term gains but rather
disappointment on the horizon, slowed adoption/research and of course
additional engineering work to remove it.

So why have Chrysalis pt 1 at all? Is it worth it for a few TPS hype tweets?

Independent of my opinion on Phase 1, I think the changes planned for Chrysalis
Pt. 2 are good & important and will fix some of IOTAs more serious current
problems. I also want to stress that I don't consider this the fault of individual
engineers. The actual design on it's own and the implementation are fine. But I
don't think management should have prioritized this project over Phase 2 or
Coordicide.

There is some irony here, in that I have previously joked that "all of CfBs
ideas that distinguish IOTA from other cryptos are being undone with Chrysalis,
except the DAG & feelessness". Well, the DAG is *effectively* being removed
with Chrysalis pt. 1, even though it's not explicit. I'll write an article
on feeless some other time ;)

