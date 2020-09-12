---
title:  "Hot take: Mana specs: I have questions"
description: Mana is Coordicide's main sybil protection mechanism, 15+ months in the making. The specs that were just released create more questions than answeres. Way more. Here's a few.
date: 2020-09-11
---

# Hot take: The Mana specification is bad, and the IF should feel bad


**TL;DR**: Mana is the main Sybil protection mechanism for Coordicide, 15+ months
in the making. [The specification](https://github.com/iotaledger/goshimmer/blob/22f88a897fd6729980fdca3f08f16bb71b0d3caf/docs/001-mana_proposal.md)
answers virtually no questions, and
(explicitly and implicitly) opens up dozens more.

IOTA managed to turn the "one, constant mana" from the whitepaper into 4
different manas, in groups of two, leaving it completely unspecified what you
would use each one for and how to combine them.

*Updated 2020-09-12: Added section at bottom to address criticism*

**Note**: I think lzpap did a good job at collecting the available data & deriving
an implementation from it. Even though I do have issues with the implementation
design (Why the fuck does Get\*() cause a ManaUpdated() event? Leak
implementation much?), 99% of criticism is with the state of the research.

**Note 2**: I didn't want to spend too much time on this, so this is a list of questions
in bullet points. It's barely more than a rough draft, but it's already about
as long as the "high-level" design section in the specs.  You're welcome. Some
of these questions I believe are fundamentally very, very
hard to solve and are completely ignored.


## The specification adds complexity without telling anyone why.
- Why do you need consensus Mana *and* access Mana? Why can't you use the same one?
- Why do you need EBM 1 *and* 2? (The constant *and* the decaying one)?
- It says that you should combine them "in yet unspecified way". Any reason why anyone should do this? Any examples for problems you could solve with some combination?
- Why do you need a second timestamp field in the transaction?
    - There is also an *implicit third* timestamp, the confirmation time on the node.

## The specification leaves fundamental questions completely unaddressed
- How long does it take for Mana to decay to 50% of it's value? A minute? A week? A year?
    - If it is, say, 1 Week - does this work with IoT devices that send sensor data every minute? Does this work with wallets holding for years?
    - If it is in the order of Minutes, how do you make sure that attackers don't abuse propagation delays to create inconsistent node-subjective views of the mana state?
    - If it is in the order of Months, how is this different than PoS/Delegated stake?
- If mana is applied after a transaction is solid, and nodes need some mana to send transactions, how does a node "dig itself out of a zero-mana hole"?
    - If a new node comes online and starts with zero mana?
    - If the network (or neighbors) are near capacity, and have elevated mana requirements for rate control?
    - If the answer is "all nodes have a small minimum effective mana that
      allows them to send a small amount of TPS", then what stops me from just
      generating new NodeIDs for every transaction? How is this a Sybil protection
      mechanism?
- How exactly does `pending Mana` relate to `Base Mana 2`? I'm assuming it's the same? Maybe?

## There are obvious DoS vectors that are not mentioned
- Mana dusting on non-existing node-IDs (making maintaining the mana vector expensive)
- Mana flipping with subsequent transactions

## There are attack vectors through circular dependencies with other modules
- e.g. Rate-limiting
    - The Mana of node X effects how likely (or quickly) nodes are willing to accept transactions issued by X - so I can *actively* control how fast nodes are propagated through the network.
	- What if I flip back and forth node X's mana in dependent transactions?
	- What happens then with all the other transactions that are sent through X?
	- What happens with all the transactions that depend on the flipping transactions?
	- Think [this video](https://www.youtube.com/watch?v=Q78Kb4uLAdA), but with multiple intersections/branches, and multiple chokepoints that keep appearing and disappearing all the time.
- Kicking nodes when they are down through rate limiting, autopeering/gossiping
    - As pointed out above, it might be really hard to dig yourself out of a mana-hole once you're in it.
    - So what stops an attacker from picking a random node with poor mana, and
      just DDoS that node (or stop forwarding transactions that assign mana to
X) to the point where that node is stuck in the mana hole?
    - What stops the attacker to repeat the same thing with the next poorest
      node Y?
- e.g. Tangle-state
    - Messages depend on Mana. Transactions issue Mana. Transactions are contained in Messages.
    - It's another instance of a negative feedback loop, once you're in a hole
      it's hard to get out again.

## Unclear Terminology
- The spec claims

  >  "Mana is essentially the reputation score of a node in the IOTA network"

  but the term "reputation" has
  previously been used to describe an aggregate value for 'what one node thinks
  of another', of which *mana* is one part, but which can additionally be slashed
  (e.g. bad berzerk detection), and contains non-verifiable parts too (e.g. DOS).
- 'Effective Base Mana 2 for the Consensus Mana'. Seriously?
- There are now two timestamp fields, one for the message and one for the
  (encapsulated) transaction. They are both called timestamp. They are allowed
  to be different by a few seconds, for unclear reasons.


## The specification doesn't actually tell us what problems it's trying to solve, apart from a vague "reputation score"
None of the above matters, because at the end of the day:

- Decaying mana was supposed to protect IoT nodes from someone willing to spend a few thousand $ for an attack. Does this do that? How?
- "The scope of the first implementation of mana into GoShimmer is to verify
  that mana calculations work" - but Mana is a Sybil protection mechanism. The
  scenario is fundamentally different when Mana is in monitoring mode vs. when
  it's actually influencing the system. Attacks on it can only happen in the
  latter, but the test is only for the former.
  
  *I agree that this needs to be tested somehow*. But the document uses this as an excuse to "fill in the blanks later".

  You can **only test the steady state** behavior of mana. You can not test
  whether mana actually withstands attacks. Virtually all discussions on attack prevention
  need to happen theoretically, in the specs, and this is just completely absent.
  

Anyway, as trivial as it is to poke holes into this, I don't want to spend too
much time on this. Happy to dig deeper if Mark gives me EDF funds to do so.

There is a quote, "in distributed systems, i++ is a PhD thesis", and I think that
is very much true, even in permissioned systems.

IOTA is trying to build exponentially decaying systems based on multiple, rapidly changing
inputs, to be the basis for all their other modules. And after 15 months of research,
they've written down one equation. Good, but now comes the hard part, finding values and
making the calculation consistent across distributed, and potentially hostile, networks. See you in 15-50 years or so ;)


## Edit 2020-09-12: Addressing some criticism on this article

The main criticism seems to be that there are
more internal specs that are yet unpublished, which address the mentioned problems. However, the [research
update](https://blog.iota.org/iota-research-status-update-september-2020-72720fa1c032)
referred to the specification as follows:

>  This month we have worked on turning the mana research specification into a
>  more detailed
>  [document](https://github.com/iotaledger/goshimmer/blob/22f88a897fd6729980fdca3f08f16bb71b0d3caf/docs/001-mana_proposal.md)
>  that we are currently using to implement mana in Pollen. 

Notice the past tense in "turned", the "detailed", and the "currently using to implement mana".
There is every implication that *this is the spec*, and no implication that there is
any other in-progress document.

The main criticism seem to be on "iterative development", which I've already addressed in the last
bullet point, but have re-worded to be a bit more specific.

