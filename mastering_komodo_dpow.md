# Mastering Komodo Delayed Proof-of-Work

## by Duke Leto

This is a short technical document aimed at developers who are interested to learn
more about DPoW and/or want to add it to their a cryptocoin to protect
against consensus (51%) attacks. These attacks are becoming very common
against coins from large to small.

This book assumes basic knowledge of the Bitcoin protocol, Bitcoin source code,
C++ and basic compiler knowledge. No prior knowledge of Komodo is assumed, this
work contains a basic introduction of Komodo features.

# High Level Overview of DPoW

DPoW sends blockhash data from your coin, to KMD, which is then sent to Bitcoin,
providing a small coin with a small hashrate the security of BTC hashrate. This
data is embedded in OP\_RETURN metadata which Komodo Notary Nodes continuosly
send. Notary nodes run full nodes of Bitcoin, Komodo, and all Komodo Asset chains
and coins protected by KMD DPoW, and are the way data from one chain makes it onto
another chain. The author of this document helps run the [ChainStrike notary node](https://komodostats.com/node.php?id=16).


## Global Notary Node Network

In more detail, there exist a network of 64 notary nodes spread around the
globe.  They are not in any one physical location, which makes the system
resilient against a datacenter outage or natural disaster in one geographical
location. The algorithm NNs use require 13 notary nodes to come together and
agree on what the latest Merkle root and other metadata that will be embedded
into the Komodo and Bitcoin blockchains inside the OP\_RETURN data of a
multisig transaction. This means that to take down the notarization process, to
stop the data flow, one must attack and knock offline 52 different servers in
diverse locations,  simultaneously. In this case, notarization data would stop
flowing, but no "fake" or "incorrect" notarization data can be created, because
the source code of Komodo has the public keys of notary nodes embedded inside.

Only notary nodes have the corresponding private keys to make transactions that
the network will trust as valid notarizations. So DDoS attacks can only stop
notarization data from flowing temporarily, they can not be used to knock
actual notaries off-line and then impersonate or man-in-middle the process.

## Adding or changing DPoW is a Hard Fork

To be clear, DPoW are consensus-level changes to a cryptocoin source code,
they require a coordinated effort of all exchanges/mining pools/users upgrading,
since it is a hard fork. Take this into consideration when planning upgrade
timelines.

## Yearly Hard Forks

Every year Komodo does a hard fork, which coincides with electing new notaries,
which usually happens in early May. Electing new notaries means changing the
KMD source code to list these new pubkeys and removing old ones. This means
that any coin using DPoW will *also need to hardfork at a coordinated time*,
i.e.  coins using DPoW will also fork in May each year, after new KMD notaries
have been elected and the new notarization season begins.

Coins can turn this yearly hard-fork into an opportunity to make any other
non-emergency changes that would be consensus-level hard-forks. It allows
a yearly upgrade window for each coin, where a hardfork is going to happen
anyway. For instance, the size of the maximum OP\_RETURN might be changed,
or a different difficulty algorithm used (test it thoroughly first!).

Additionally, there is a preferred time of year where it makes the most sense
to integrate DPoW, essentially directly after the yearly KMD notary election
and hard-fork. That prevents a situation where adding DPoW integration in March
requires a hard-fork and then the KMD notary election requires another
hard-fork just a short-time later. Exchanges are not excited to update full
nodes and some exchanges attempt to charge for full node updates (which the
author finds scammy), so timing the integration of DPoW services can be very
strategic.

# Double Spend Attack Prevention

It is the authors recommendation that exchanges pause any transactions for
which data has not yet been notarized to the appropriate chain. For a game
allowing in-game purchases with digital currencies, it might allow transactions
up to $10 if the data has been notarized to Komodo, or no limit on transactions
that have been notarized to Bitcoin. This allows small-value transactions to
proceed very quickly (Komodo has a 1 minute blocktime) while ensuring
protection against double-spend attacks for large value transactions. This
protection comes at the cost of waiting longer, as Bitcoin has 10 minute
blocktime.

It is important to understand that one half of the process is notarization
data, which must go from COIN -> KMD -> BTC, and the other half of Double Spend
Protection is actually looking at this notarization data and deciding if it's
safe to proceed with some action. Komodo is working on an API to see if a txid
of a certain dpow-enabled coin has been notarized to KMD and BTC, so that
exchanges and other parties wanting to protect against double spends can make
informed decisions, in much shorter time periods than waiting until 120 confirmations.

Currently exchanges often wait 120 confirmations, which is well beyond the 100
block reorganization limit that is part of the Bitcoin protocol in many
cryptocoins.  With two minute blocktimes, that is 4 hours to wait, instead of
waiting a much shorter time and being able to ask an API if a transaction has
been included in a notarizion to Bitcoin.

So there is a trade-off: simplicity and waiting longer, or integrating with a
Komodo API (more work) and having more precise and real-time knowledge of when
a txid can be considered "safe" from a double spend.

The plan is that any party can and should run their own instance of this API,
which relies on talking to full nodes of BTC, KMD and the coin being notarized.
Additionally KMD will run it's own publicly available instance, and an exchange
or other organization can decide which servers, other then their own to query.
Any difference in the response from different instances should be inspected by
a human, since it indicates incorrectly set up software or a potential attack.

The API for when notarizations have made it to the KMD chain is at
https://komodostats.com/api/notary/summary.json and the API for looking up an
arbitary txid and whether it has been notarized to Bitcoin is in development.

# Cost of DPoW

The raw cost of 64 global notary nodes making transactions roughly once per
minute (the Komodo block interval) has a cost. There is currently a one-time
yearly fee of 300 KMD and 300 COIN paid directly to jl777 to cover these costs.
To protect against future price increases of KMD (it's currently just over $1
USD), one may purchase 6300 KMD, and then use the yearly 5% interest of 300 KMD
to pay the yearly notarization costs. The 300 KMD is used the first year and
then the remaining 6000 KMD will generate 300 KMD per year via 5% interest.

NOTE: These prices are very subject to change and will most likely increase as
we have a fixed number of resources to help with integrations.

Note that since block 1,000,000 of KMD, funds must be moved once per month to
get the full 5% interest. Interest stops accruing after one month for UTXOs
created after block 1M. UTXOs from before still accrue interest without needing
to be moved.

Also note, that if you decide to migrate your existing coin to a Komodo Asset
Chain, there are no integrations needed or integration development costs. Just
the cost for notarization transactions. This can be accomplished by doing an
airdrop from your current chain to a new KMD asset chain, where people use their
private keys to unlock the funds they owned on the original chain. This can be
done by the project itself or Komodo can help with this process.

# Integrating Komodo DPoW Into Your Coin

Let us assume Alice is the lead developer of a cryptocoin based on the Bitcoin
source code. For example, since Litecoin and Zcash are Bitcoin forks, any LTC
or ZEC forks count as Bitcoin forks, too. The version of Bitcoin internals that
the coin forked is important, this determines which header file that will be
immediately compatible or at least a close starting point for development work.

For example, if your coin is based on Bitcoin 0.15.x, there already exists a
header file that matches that version of Bitcoin internals exactly. The GAME
coin was the first external coin to start using dPoW and [this commit](https://github.com/gamecredits-project/GameCredits/commit/e65fe302111408c02d2bf7e286205d4273fa0fed)
shows how they integrated. This was first created by the author of Komodo,
jl777, and then the author of this document ported that header file to Bitcoin
0.11.x internals, which Zcash and all Zcash forks have as their internals. So
if your coin is a Zcash fork, you should use the
[komodo\_validation011.h](https://github.com/MyHush/hush/blob/e529ad8d35d4dabf66fa10982e056149414f179b/src/komodo_validation011.h)
header file. If your coin has SegWit support, or is 0.14-0.16 internals, the
[komodo\_validation015.h](https://github.com/jl777/chips3/blob/dev/src/komodo_validation015.h)
file is most likely the best starting point to integrate.

NOTE: The 0.11.x header file has some improvements which have yet to be ported
to the 0.15.x header file, specifically dynamic generation of Notary pubkey
address, which is currently hardcoded in that header.

For coins with older internals, such as BTC 0.10.x and earlier, use the BTC
0.11.x as a starting point, so less changes are required.

If your coin has made various internals changes and selectively added BIPs
or other internals changes, you will most likely need to make various changes
specific to your coin.

## Calculating Notarization Lag

We will use Hush, a Zcash fork, as an example to estimate notarization lag,
i.e. the time it takes for notarization data to make it from one chain to
another. To estimate the "worst case" time it takes notarization to get to
Bitcoin, we can simple add up all the blocktimes involved, so we will be adding
blocktimes of

    HUSH + KMD + BTC = 180s + 60s + 600s

which is 840s or 13.5 minutes for blockhash data from Hush to be notarized all
the way to Bitcoin. This is the time it would take such that an exchange could
ask the Bitcoin blockchain if a given txid has been notarized.

Note that the above block-times are "worst case" under normal network
conditions, i.e. a transaction is made a millisecond after a block, so it has
to wait the entire block interval on HUSH. And then is unlucky enough to have
to wait an entire KMD block interval and an entire Bitcoin block interval.

It's possible that a bug or attack or large difficulty change makes one block
much longer than the average block interval time, which would increase the
time it takes for notarization data to make it to BTC. This is why a specific
time cannot be used to decide it's "safe", very often 13.5 minutes would be enough
time for Hush block hash data to get to the Bitcoin chain, but there could
be times when it is not.

On average, users will only wait half of each coins block time, which
means 13.5/2 = 6.75 minutes for Hush blockhash data to be written to Bitcoin.

## Is My PoW algorithm compatible with DPoW?

There are many PoW algorithms to choose from these days and thankfully
the DPoW algorithm is blissfully ignorant of all these differences. It
does not care if something is SHA256 versus Equihash, all it sees is a
256 bit block hash, i.e. a 256 bit integer. It's just a number. Any
coin using a 256 bit blockhash is compatible, which includes most
known coins.

## Integrating a Zcash fork

For the specific case of adding DPoW to a Zcash fork, the 0.11.x header file,
first implemented for Hush, should be used. Zcash forks are characterized by
being pre-Segwit and having some differences in how UTXOs are stored in LevelDB
compared to post-Segwit coins.

## Integrating a Litecoin fork

For Litecoin forks, the version of BTC internals inside is what makes the biggest
difference. Being pre-Segwit means using the 0.11.x header file as a starting point,
and post-Segwit can use 0.15 header file.

## Integrating a non-Bitcoin-derived coin

It will be most likely be challenging to integrate a coin not derived from
Bitcoin source code but we are eager to see if it's possible. The currently
available 0.11 and 0.15 header files can be used if the project is in C++ but
if the coin is written in another language then those header files must be
ported to that language first.

If your coin reimplements Bitcoin protocol from scratch, then it may still be
compatible. Please join #developer on Komodo Discord to ask more questions.

### Integration Procedure

This is a high-level procedure of how to integrate an arbitrary coin. It will
not describe every single step but it will be applicable to most situations.

* Create a new branch for dpow work
* Copy the correct header file (komodo\_validation0XX.h) to the `src/` directory of your coin
* Copy the [komodo\_rpcblockchain.h](https://github.com/jl777/chips3/blob/dev/src/komodo_rpcblockchain.h) to your `src/` directory. This file will not need
  many, if any, changes

* If you are already using BTC internals 0.11/0.15, skip the next step
* Make any API changes required to get header files compiling on your
    specific version of BTC internals
* Make sure your code dynamically generates addresses (0.11 does, 0.15 does
    not yet)
* Do a fresh sync of 2 new nodes on the branch with dpow code
    * One node with have txindex=1
    * One node will not have txindex
* If both nodes can sync from scratch with no bad errors, then you are most
    likely done with integration!
* Contact Komodo for the next round of instructions, which is Notary Nodes
    testing your code

### Notary Node duties when adding a coin

This is a non-exhaustive list of things that notary node operator will do
to enable the new coin on production:

* Ask decker to add coin to [Notary Addresses](https://deckersu.github.io/notaries_addresses.html)
* Verify notary addresses match those generated/hardcoded in code
* Install full node software
* Compile on correct branch
* Fully sync full node with txindex=1
* Import their address to the new coin via a WIF generated from pubkey
* Run full node with -pubkey option
* Receive coins funds to be used in notarization UTXOs
* Do a round of testing with enough NNs to do a dpow round (a min of 13)
  to verify code more
* Set up automated procedures specific to each coin, such as UTXO splitting/etc

## Adding some RPC methods

You will need to add a few RPC methods to your coin so that will allow
software to ask the node various questions about Merkle roots of Merkle roots
(MoMs). Crosschain proofs are Merkle roots of MoMs and are called MoMoMs, this
feature is still in development.

### calc\_MoM height MoMdepth

Calculate a MoM (Merkle root of block Merkle roots)

Arguments:

height: Height of first block MoMdepth: number of blocks to include

Eg:

```
calc_MoM 100000 20
Response: { "coin": "DATACHAIN" , "height": 100000 , "MoMdepth": 20 , "MoM": "53c234e5e8472b6ac51c1ae1cab3fe06fad053beb8ebfd8977b010655bfdd3c3" }
```

# More coming soon


# Supporting This Document And Author

The author is currently a Komodo Notary Node operator and Komodo Core Developer
who relies solely on cryptocoin income. If this document has helped you and
you would like to support future updates, you can send donations to
the KMD address RBSEv7nJ1wciriVyLFWotQ8tBvS2rKwYtz . Thanks!

