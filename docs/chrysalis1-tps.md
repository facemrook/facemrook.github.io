---
title:  "IF exaggerates Chrysalis TPS by 10x"
description: IF claims that the last stress test of Chrysalis Phase 1, the network achieved 1000 TPS. In reality, this number is closer to 70-100. The IF knows this.
---

# IF exaggerates Chrysalis TPS by 10x

TL;DR: IF claims that the last stress test of Chrysalis Phase 1, the network achieved 1000 TPS. In reality, this number is closer to 70-100, **and the IF knows this**.

Tweet: 

[https://twitter.com/iotatoken/status/1296112002155393024](https://twitter.com/iotatoken/status/1296112002155393024): @iotatoken claiming 1000 TPS.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">It&#39;s official - The <a href="https://twitter.com/hashtag/IOTA?src=hash&amp;ref_src=twsrc%5Etfw">#IOTA</a> network now supports over 1000TPS! With this substantial increase, nodes can sustain higher transaction volumes &amp; more transaction intensive applications. We’re excited to see the applications &amp; use cases enabled with this new throughput! <a href="https://twitter.com/hashtag/Chrysalis?src=hash&amp;ref_src=twsrc%5Etfw">#Chrysalis</a> <a href="https://t.co/RtVbiytCO3">pic.twitter.com/RtVbiytCO3</a></p>&mdash; IOTA (@iotatoken) <a href="https://twitter.com/iotatoken/status/1296112002155393024?ref_src=twsrc%5Etfw">August 19, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

# IOTA defines "TPS" differently than everyone else

## 1000 Transactions are really 250

TL;DR: You can't compare IOTA "Transactions" with other DLT's transactions. Bundles are comparable, and 4x slower.

- What is typically called a transaction in DLTs is *moving tokens*. E.g., Bitcoin does some ~3.7 TPS, which usually include one or two inputs and one or two outputs. Same with Ethereum does the same at about 15 TPS.
- IOTA can bundle multiple "IOTA Transactions" into a "Transaction Bundle". Zero-value transactions (probably?) only require a single "IOTA transaction" in the bundle, but value transactions require more than that.
- **Thus, to compare IOTA against other DLTs, the comparable metric is Bundles Per Second, not IOTA-Transactions per second.**
- But this is not what IOTA does: IOTA counts individual "IOTA Transactions" ("Transaction fragments", or in Bitcoin terms, individual inputs (twice) + individual outputs). [see source in appendix]
- The concept of an IOTA-Transaction is an *implementation detail* that people outside of IOTA do not know, or care about. The **effect** - some tokens are transferred - is what matters.
- The difference is significant. Because inputs are counted twice (for the signature parts), there are *at least 3, typically 4 or 6 "IOTA transactions"* in each bundle. "3" only happens when all funds from an address are sent to a different address, with no remainder, which would be somewhat rare.
- I would like to stress that you *could* count individual inputs & outputs in Bitcoin as "transactions", but they *don't*.
- Assuming 4 on average, this would make 1000 TPS more like 250.

## "Well why does it matter? IOTA counts differently"

- The outside world defines a Transaction as "moves tokens"
- This was not an internal post, IOTA *tweeted "1000 TPS" to the outside world*. There was no attempt of making the distinction clear.
- From the tweet above, a *reasonable person in the target audience of that Tweet* would assume that IOTA moves 300 times more tokens per second than Bitcoin, 70 times as many as Ethereum, or about the same as Nano. And this is clearly not true.
- The IOTA foundation knows that this distinction exists (see below), which makes this tweet actively misleading.
- Also, if "we count differently" is an argument, the next DLT could just count their TCP/IP packets and re-label them as "Transactions" (this is not as far-fetched as it sounds - TCP packets have similar guarantees than zero-value transactions, see below)
- Added 2020-08-22: The IOTA-community internal discussions following this article made it clear that this was not a well-known fact. The discussion in #general alone included multiple foundation members (Navin, Mark, Luca, muXxer) and went on for hours; well known community members weren't aware of this either ([tweet1](https://twitter.com/dennisnagpal1/status/1296899495230418945), [tweet2](https://twitter.com/durerus/status/1296882900189818880)).
**A large chunk of IOTAs own community believed that the 1000 TPS was refering to 'value TPS' after reading @iotatoken's tweet. This is the definition of misleading.**

# Nodes were falling off the network at 20-40% of max TPS

TL;DR: Raspberry Pis, explicitly recommended by the gohornet team, were falling off the network at 150-400 "TPS" (or, 1/4 of those, 35-100 value TPS).

The first two sentences on the gohornet page read:

> HORNET is a powerful, community driven IOTA fullnode software written in Go. It is easy to install and runs on low-end devices like the Raspberry Pi 4.

The implication here is that a node running on a Raspberry Pi 4 can be a *fullnode in the IOTA network*, and the tweet specifically stated that the *network* reached 1000 "TPS". But they weren't, and the IF knows this:

![/chrysalis1-tps/Untitled.png](/chrysalis1-tps/Untitled.png)

![/chrysalis1-tps/Untitled%201.png](/chrysalis1-tps/Untitled%201.png)

![/chrysalis1-tps/Untitled%202.png](/chrysalis1-tps/Untitled%202.png)

So this means that raspberry Pis were dropping out of the network (loosing sync) at 150-400 **messages** per second (so about 35-100 transactions/sec). (I don't know whether they were able to recover below that threshold, or whether they went into cascading failure mode).

So, either the network does 1000 "TPS", or Raspberry Pis can be part of the network. Can't have it both ways.

# More broken stuff

## Zero-Value transactions are not transactions (and the IF knows that)

- Zero-value transactions are content-addressible storage. There is no semantic validation & no authentiticity checks. The only thing that's partially guaranteed is delivery - somewhat similar to TCP/IP packets. As they don't move transactions, they can be re-ordered in any way (in particular after the white-flag upgrade, where you can even attach to conflicts).
- The IOTA foundation knows that calling these objects "Transactions" is problematic. Which is why "messages", and "messages per second" is already being used in goshimmer.

## White Flag gave you that speedup, and it will be removed again

- White flag seems to be the main driver behind the speedup, as it simplifies GTTA (you can now attach to conflicts, no need to do conflict checks). This turns IOTA into a giant mempool but that's beside the point.
- White flag requires checkpoints (as in, blocks in a blockchain, issued by the coordinator). This feature will have to removed again before coordicide (implied in the [specs](https://github.com/thibault-martinez/protocol-rfcs/blob/rfc-white-flag/text/0005-white-flag/0005-white-flag.md), explicitly stated as "valid for the pre-coordicide era" by David in [this blog post](https://blog.iota.org/chrysalis-b9906ec9d2de) )
- That means that the speedup will likely disappear again. Post-coordicide, you'll have to reattach & promote again.
- Side-note: This somewhat invalidates the idea that companies "can start building and evaluating applications on top of IOTA". The performance characteristics & semantics of referencing transactions will change again.

## Other

- "But chrysalis phase 2 will remove bundles" - Yeah. But the tweet was about phase 1.
- "zomg r/cryptocurrency hates & downvotes IOTA" - No, they just realized that virtually all IOTA PR falls apart when you actually ask questions. Happened prominently above (Coordinator, Jinn, Qubic, We Did It), and still happens.
- Maybe *someone* should be held accountable for constantly misrepresenting the truth to the investors, just saying. I'm non-invested, I'm just here for the tech.

I also want to make clear that I'm only commenting on how the IF has handled this, and I'm not trying to disparage the gohornet team. Hornet seems like a pretty solid piece of software, a significant improvement over IRI. Even at 1/10th of the advertised TPS, I think this is a pretty good result for Raspberry Pis.

# Appendix

## TPS counting method

![/chrysalis1-tps/Untitled%203.png](/chrysalis1-tps/Untitled%203.png)

![/chrysalis1-tps/Untitled%204.png](/chrysalis1-tps/Untitled%204.png)

![/chrysalis1-tps/Untitled%205.png](/chrysalis1-tps/Untitled%205.png)

![/chrysalis1-tps/Untitled%206.png](/chrysalis1-tps/Untitled%206.png)


Edit: Because I keep getting told that "you just screenshotted a fraction of muXxers message", here is the full thing. Spoiler: I've addressed all these points above already, the full quote has no impact on the key argument(s).

![/chrysalis1-tps/Untitled%207.png](/chrysalis1-tps/Untitled%207.png)

The two "additional" arguments here are

- *"but zero-value transactions would only be one per bundle, so BPS would be misleading true"* - well this is true, but it's just another way of looking at the same argument. When a DLT talks about "TPS", people expect *"funds moved per second"*. If BPS doesn't capture that, and "IOTA-TPS" doesn't capture that, that needs to be made explicit. if you call your metric TPS, you need to publish something that's comparable, or state that it's not comparable. Everything else is just plain misleading. (Why not just count *bundles-with-value-transactions* as a separate metric, and publish that? Problem solved.)
- *"With Chrysalis pt2 this will no longer be an issue"*. As stated above, the tweet is about part 1. Part 2 is months out. Nobody knows how that will perform (even though I expect it to perform better).
