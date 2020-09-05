---
title: Zero-Value transactions - Fun but worthless
description: Zero-value "transactions" (messages) are fun for hobbyists, but give no guarantees to developers & industry partners, making it a bad platform to build production systems on.
---
# Zero-Value transactions: Fun but worthless

## Rough draft. Discussion in \#serious\_tech\_talk

TL;DR: Zero-value "transactions" are fun for hobbyists, but give no guarantees
to developers & industry partners, making it a bad platform to build production
systems on.

Note: Per the reaction on a previous article, I'm always happy to consider
feedback & will update the article where I'm wrong. However, I do only consider
things that are actually specified. Off-hand ideas thrown around on Discord are
hard to verify, hard to analyze & hard to argue with.

# Zero-Value Transactions are unreliable

Popquiz:

If you write a temperature reading to the tangle and broadcast it to a node of
your chosing, what are your guarantees about the data?

1. Are they guaranteed to be in the tangle?
2. What if the node retries (reattaches/promotes) for you?
3. After you've seen them being confirmed, are you (or other people) guaranteed
   to be able to read them in a month? a week? immediately?
4. Are they immutable?

For any other database systems (e.g. AWS DynamoDB), the answers are **Yes, Not needed, Yes/Yes/Yes** and **"when wanted & legal"**. For IOTA, the answers are **No, No, No/No/Maybe** and **in a problematic way.**

## 1. Data might not persist in the Tangle, even if your node says so

If you commit to a single node, and get an OK back from the RPC, there are still plenty of error cases which can result in your transaction being dropped.

1. Your transaction might simply be voted out (because one of it's dependents
   conflicts[^conflicts]. If gossiping is done over UDP, and you're having a
   bad minute, no neighbors (or too few) might actually hear about the
   transaction.
2. The node might crash at any point and simply drop your transaction. If it
   takes a very (?) long time, the node might just prune the transaction before
   it is even sent[^pruning].

At the time of writing, [thetangle.org](http://thetangle.org/) shows a 97%
confirmation rate, even with the temporary White Flag boost. This means 3 out
of a 100 writes fail. For comparison, AWS DynamoDB will start [refunding
money](https://aws.amazon.com/dynamodb/sla/) below 99.99% ("standard") and
99.999% ("global") availability.

Production-ready database system create reliability from unreliable components
by making sure the data is properly replicated, persisted to disk and (in case
of strongly consistent systems) that consensus exists that the write should be
applied. This normally happens in a few hundred miliseconds, **before the
client is given an OK**. IOTA can't guarantee any of these, by design.

[^conflicts]: I know that hans has mentioned an “approval switch” to make this
    problem less bad. This seems to break merkle proofs & the DAG approval
    structure (i.e. everything in The Tangle whitepaper). Given that this has been
    an off-hand comment on #tanglemath, and i’ve seen no actual specs for this, I’m
    going to assume that this is something in Hans’ head which may or may not
    actually work out)

[^pruning]: Everyone has an approximate vague concept of how long components - hard
    drives, SSDs, CPUs, RAM, PSUs - last, usually derived from personal experience
    using PCs. It is important to note though that *load* is an important factor in
    life-time, too. It is already well known that SSDs degrade with the amount of
    writes, and similar effects (although not as extreme) happen with other
    components, too. Have a look at your Task Manager and check your utilizations,
    and compare that with the 100% goal that data center operators strive for.

## 2. Nodes may prune at any time.

- Nodes might prune at any time. The default in Hornet is
  [7 days for Mainnet](https://github.com/gohornet/hornet/blob/c82de7ec57b5aed7528ffc8ed325430ec619b096/config.json#L70),
  so after only a week a transaction is likely to be gone.
- Nodes may actually **have** to prune, for GDPR reasons (see blow).
- Nothing stops node owners to configure the node to just update their ledger
  state and *immediately* prune.
- Nodes may also prune value transactions, they only need to retain the ledger
  state. In fact, Popov has already
  [hinted](https://iota.cafe/t/every-1i-in-a-single-address/368) that low-value
  transactions don't necessarily need to be processed (verified, attached to,
  ...) at all.

This is by no means hypothetical: An example of pruning affecting a production
system can be seen on
[http://transparency.iota.org](http://transparency.iota.org/) (which might just
be the *only* production system using IOTA): The "transaction" links (bottom
right in each card) for grants older than 7 (?) days show no data ([example
from the Hornet Phase 2
grant](https://utils.iota.org/transaction/TBNWXUM9MNIS9KINDSGZUIUMYC9VKCDKDTEIEAQJVXYLKBWHXAOBWBIAHFTIKVFZSIVAUDJLVCPU99999),
14 days old at the time of writing). The site,
[utils.iota.org](http://utils.iota.org/), is not broken though: heading to
[thetangle.org](http://thetangle.org/) and trying one of the recent ones on
[utils.iota.org](http://utils.iota.org/) does work for me.

## 3. Data is immutable (until pruned), but that's hardly a good thing

- Immutable data is not necessarily good (GDPR-compliance)
- And yes, while most other blockchains suffer from this too, most other
  blockchains are not specifically *designed* to store data. You may get a few
  bytes of payload, not kilobytes. This is also one of the biggest problems of
  society2, which society2 seems to completely ignore and/or make the
  problem of the node owners.
- A society2 lead has literally stated that GDPR will be changed to accommodate
  society2, which - sorry - is the dumbest thing i've ever heard. Weaken one of
  the most important privacy regulations to accommodate a single social network.
  Not to mention that there are no GDPR-equivalent regulations in multiple other
  countries and US States.

# Other statements that don't hold up

## Messages are verified

The argument here is that the Tangle does create validation through the Merkle
tree (some form of verification).

The first thing to look at is what is actually verified.

- Messages are not automatically "more authentic" just because they are
  included in the tangle. Authenticity is created through digital signatures
  (e.g. through MAM, Streams, or any other scheme), but these are created on the
  sender side and verified on the recipient side - the tangle just transmits and
  briefly stores the data.
- The term "DID", or "Digital Identifier", gets thrown around a lot in the IOTA
  community, but this is the same as above: It is something that can use the
  tangle, but just as well any other storage system. In fact, the only time that
  the W3 spec even talks distributed ledgers specifically, it is [to warn people
  about the problems with distributed ledgers and
  GDPR](https://www.w3.org/TR/did-core/#keep-personally-identifiable-information-pii-private).

What the tangle (and other blockchains) actually does is called "Trusted
Timestamping". In effect, it verifies that some piece of data existed at a
specific time. The timestamp in each transaction becomes part of the hash,
which then gets referenced by other transactions etc.; standard merkle tree
stuff.

But using the tangle for timestamping is not straight forward - the tangle is
mainly built for feeless microtransactions, not timestamping. Two large
problems:

### Merkle-proofs require more than just than the transaction

Without going into too many technical details, due to necessary optimizations
[(see Hans' post in #serious\_tech\_talk for
details)](https://discord.com/channels/683820557736607745/705714758497730561/719457392160407683)
you **cannot actually prove** the statement "is transaction X in the tangle":

Due to the way you need to optimize merkle proofs, you need to observe the
tangle for a while after you published the message. In order to prove inclusion
in the tangle, you need to provide not only the message hash, but also a part
of the DAG (minutes or hours worth of tangle activity).

### Positive proofs only

You can only trust "postive" proof-of-inclusion results:

- Nodes can prune their tangle at any time, so you can get false negatives.
  Unless you store your data in a (trusted) permanode, but operating one would
  nullify all the benefits of using merkle proofs.
- Nodes can just decide to not do the work and just reply "no". Unless you
  trust the node you query (in which case you don't need the tangle).

In other words, if you own a message that you'd later like to verify exists in
the tangle, you might be able to do so, but not in a trustless manner.

### Free Trusted Time-stamping in split-seconds is already a thing

Instead of jumping through all the hoops above, why not just use an rfc3161
timestamping server? [Plenty of free ones are
available](https://gist.github.com/Manouchehri/fd754e402d98430243455713efada710),
which you can all use in combination if you like, and if your threat model is
"Multiple CAs, Universities & Apple are compromised", you have different
problems anyway (and certainly should not be relying on experimental blockchain
technology)

## "If you need persistence, you can run a permanode"

First of all, I don't think there is a single use-case for zero-value
transaction (in a production environment) that does *not* require at least
*some* form of reliable persistence. But permanodes do a poor job at this:

- Running a full permanode would be equivalent to running a node without
  pruning. But now you are paying for a database to store a full copy of the
  full tangle, in addition to running the node[^costs] . Also, this only solves *problem
  (2)*. In particular, as soon as you have to delete an entry from your permanode
  (e.g. when legaly required for GDPR or CSAM), you're turning it into a
  selective permanode and lose the verifiability.
- Selective permanodes solve *problem (3)*, but they are weak at *(2)*. The only
  thing you can reliably query from selective permanodes in a trustless manner
  are merkle-tree proof-of-inclusion queries, with the caveats mentioned above.
  For queries on secondary indices (like "all transactions with Tag = "..."), you
  need to trust the node - there is no way to verify that the result is complete.

Neither of the two permanode types solves problem (1.) - it is unclear whether
a transaction you send will actually arrive at the database-backed permanode.
And since the fullnode will eventually "degrade" into a selective node for
legal reasons, and selective nodes require trust, the question then becomes -
why not skip the tangle & write straight to a regular database?

[^costs]: I should point out that
    running databases for production workloads costs actual money. AWS generated
    77% of Amazon's profit in 20Q2. Companies don't run their business off cheaper
    consumer-grade hardware (or even Raspberry Pis) because, at the end of the day,
    the additional reliability makes it worth it.

## "The tangle is not a database anyway"

In that case, the remaining use-case is as a "communication system". But

* As stated above, the Tangle has poor reliability - why put it in between you
  and your database in the first place?
* Systems that are significantly more performant, significantly more reliable
  and way cheaper to operate already exist. ZMQ and co. come to mind. All major
  cloud providers offer pubsub solutions, too. Heck, even BitTorrent & co fall
  into this space.
* If you need a database on the backend anyway, and you need everyone to
  be able to read it, why not open up the read ACLs to the public?
  Even if you need more advanced features, like a simple REST API Gateway with
  basic rate limiting etc. - *cost per TPS* on EC2/Lambda should be a fraction
  of that of running an IOTA node.

The Tangle doesn't really add anything as a communication system, it only makes
a hard problem worse.

# Summary

There are other problems here, that are similarly problematic, like

- is there a sweet spot between "how much data do you need to make it useful"
  ("Big Data") vs "how much data do you actually want in a DLT"
- do zero-value transactions actually solve the hard problems in data
  acquisition (or is it data quality / input errors / cleaning,
  legal&compliance, ....), or does it just add another one. The ["smart
mango"](https://medium.com/@kaistinchcombe/decentralized-and-trustless-crypto-paradise-is-actually-a-medieval-hellhole-c1ca122efdec)
  article is a recommended read on that subject.
- How to add & revoke access to existing data. This is pretty much unsolvable in a DLT, even though Streams implies that it is - I might tackle that in a separate article later on.

I might address those in a later post.

I am aware that you can make arguments (for a post-coordicide world) that
"it is decentralized", and that is certainly a valid argument. The question
is whether users, developers are willing to put up with the above problems,
just to have decentralization. After all, some IF leaders have pointed out
that industry partners do not actually care about the Coordinator that much.

The tangle is a fun platform to experiment on and build prototypes, but it is
useless when you're trying to build actual applications on top of them.

Happy to hear your feedback, feel free to send me an email and/or twitter.
