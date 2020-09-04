---
title: IOTA's colored coins are repeating BTC's mistakes
description: IOTA are adding a "second" smart-contract system to support their tokenized assets. This is unfinished, unspecified, and unnecessary.
date: 2020-09-04 
---

# Colored Coins: Repeating BTC's mistakes

## This is a draft. The discussion is ongoing in #serious\_tech\_talk

TL;DR: IOTA are adding a "second" smart-contract system to support their colored coins. This is unfinished, unspecified, and unnecessary.

Note: This is a rather minor point in the grant scheme of things and
likely not as big of a problem as some that the other posts descibe. If you're
not interested in details, the extended TL;DR is "IOTA hyped that 'tokenized
asset' support landed in goshimmer, but by any definition of "tokenized
assets" it is very much unfinished."

Note 2: The article makes the assumption that SC Quorums outlive tokens
(unlikely), or (more likely) that there will be some kind of migration strategy
to new quorums.

# What are IOTAs colored coins?

- Transactions get an additional, 32-byte "color" field.
- If you want to create a color, you set this field to a special value when issuing a transaction. The color of the output is now the hash of that transaction. You only have limited control over the hash, and you can't specify arbitrary colors.
- Color is propagated along with the tokens (from that output on).
- You can remove the color ("burn") the token.

That is all there is in colored coins right now. It should be noted that this
is a far stretch from even a ERC-20, the only (equivalents of) [it’s 11
methods](https://eips.ethereum.org/EIPS/eip-20) are `balanceOf()` and `transfer()`,
via the standard IOTA mechanisms.

Hans has mentioned in #serious\_tech\_talk that there are some ideas for
specialized opcodes to deal with colored coins, but these are not specified
anywhere (to my knowledge). It is, IMHO, a bit early to call the current state
of specifications/implementation [“support for tokenized
assets”](https://blog.iota.org/introducing-pollen-the-first-decentralized-testnet-for-iota-2-0-349f63f509a1).

# **You need Smart Contracts for Colored Coins**

IOTA’s "colored coins" approach might sound familiar: People tried an approach
with similar functionality, 6-7 years ago, on Bitcoin.

There are multiple different approaches listed on [the Bitcoin
Wiki](https://en.bitcoin.it/wiki/Colored_Coins), but generally: they would tag
transactions with a special value (abusing the `OP_RETURN` field). Bitcoin nodes
would ignore these treat subsequent outputs as regular transactions, but the
Smart-Contract support in wallets would enable semantic meaning & smarter
features. In some approaches, additional opcodes encoded in the transactions
enabled more complex logic, in other approaches the logic would be stored
completely externally.

These approaches are virtually all dead, and in most cases, have been dead for
years.

The main problem seems to be that bitcoin required the actual contract code
execution to be off-ledger; whereas Ethereum explicitly stored the code on the
Ethereum chain itself - and had full support for handling & managing
(turing-complete) complexity (using gas). This made flexible logic (even simple
stuff like total token supply, who can transfer to whom, "tokenization rules",
or even "what is the name of this token") explicitly a part of the consensus
model. In Ethereum, a smart contract is able to modify what a user can do with
their token(-ized asset), whereas in bitcoin the nodes treated them just as
normal funds. This, along with solidity as a more expressive language, has
paved the way for ethereum’s success as a smart contract platform.

## The problem with opcodes

While opcodes seem to be essential to IOTAs’ “tokenized assets” concept, I have
not actually seen a *specification* for opcodes yet[^1]. I’m not going to speculate
on the actual opcodes, and will rather make some high-level comments here.

In essence, opcodes add some core smart contract functionality to the *protocol
layer (base layer)*. The base layer would need to be largely immutable once
coordicide happens and industry partners begin writing software for IOTA, and
especially when integrating IOTA into (hard to update) hardware. If you are
changing what is considered "consensus" (e.g. adding/removing opcodes), this
needs to be rolled out **everywhere**, or risk that non-updated devices form
an incompatible fork.


The main design question would then be one of scope & feature set.  *Removing*
opcodes will be nearly impossible, because of the software that industry partners
build on top of those opcodes. This effectively gives the IF **one shot to get it
right** for the next few years, and if they mess it up (e.g. create a security
issue, or a way to double-spend colored coins), risk rendering the whole thing
unusable.

So should IOTA model be able to model ERC-20 tokens? ERC-721? Others? How do
you even know that those token types will work smoothly in the tangle?

If your answer is, “let’s just make it ~Turing-complete”, then you have a whole
new set of problems to deal with, the most notable two are
*  you need to be able to limit computational time (e.g. Gas),
*  you need to be especially careful that your opcodes don't introduce more problems (e.g. security issues, double spend).

In particular - even though Ethereum pulled this off - you don't want this
complexity at the base layer. I know that Ethereum has pulled this off but (1.)
they had their fair share of problems and (2.) Ethereum is not built for the IoT.
More importantly, IOTA already has a full featured, quorum based L2 SC system,
which can do the job just as well, if not better:

# **You don't need Colored Coins if you have Smart Contracts**

The code that is responsible for coloring and uncoloring tokens in goshimmer is
[only half of these
lines](https://github.com/iotaledger/goshimmer/blob/2119f233ea4c892fdd00c78bf1c1f0333a8bafae/dapps/valuetransfers/packages/tangle/tangle.go#L1368-L1387),
there is no reason this couldn’t be done in a smart contract, re-using the
“Tag” field (or message payload) instead of introducing a separate color
marker. This can achieve the same thing as the current design, while having the
mild benefit of having only those nodes process the opcodes that are actually
interested in it.

There is no specialized support in Ethereum for ERC-20 tokens, no more than
there is for ERC-721 or -223 or -777. It's simply a smart contract that you
send tokens to, and on which you can then invoke functions to further *do
things* with the token. This has been Ethereums success model, and has been
driving the majority of crypto product innovation over the last few years.

## Parallelism

I've seen the argument floating around that "opcodes enable parallelism". This
is simply not true. The same consensus rules hold for colored and uncolored
tokens, and as you need consensus on (and thus have limited parallelism on) the
uncolored tokens, the same holds for colored tokens, too.

This argument is especially weird since IOTA's smart contracts are already not
part of the consensus group (as they are run by external nodes, reacting
asynchronously to inputs). In a sense, IOTA has an interesting approach to
smart contracts that (1.) avoids requiring everyone in the network doing all
computations (2.) Is still reasonably secure, assuming Quorum selection & BLS
works as intended. 

Having two types of smart contracts, potentially
operating on the same tokens, with different consensus and concurrency models,
is a recipe for disaster. Smart contracts would be the better choice for
tokenized assets IMHO.

# Summary

This is likely not the end of the world, but it still rubbed me the
wrong way - this is adding unnecessary complexity IMHO, and I don't
think the "goshimmer supports tokenized assets" statement was
justified.  Because I keep getting told that I am "never adding
constructive criticism", well here are some suggestions:

- Don't add colored coins to the transactions & base protocol. Having two
  separate mechanisms doing the same thing is *bad*, it adds complexity and no
  value. Having one of these mechanisms in the base protocol is *worse*: The base
  protocol is the hardest to change post coordicide, it should be as simple as
  possible.
- Implement colored coins in smart contracts once SCs are ready. This allows
  you to experiment with different smart contract implementations & feature
  sets without having to modify the base protocol.
- Stop hyping “tokenized assets” until there is an actual end-to-end approach
  on how to tokenize assets, not just a way to “color coins”

TL;DR: Focus on a simple & working base layer, build things on top later.
