# Mastering Komodo Delayed Proof-of-Work

## by Duke Leto

This is a short technical document aimed at developers who are interested to learn
more about DPoW and/or want to add it to their a cryptocoin to protect
against consensus (51%) attacks. These attacks are becoming very common
against coins from large to small.

This book assumes basic knowledge of the Bitcoin protocol, Bitcoin source code,
C++ and basic compiler knowledge. No prior knowledge of Komodo is assumed, this
work contains a basic introduction of Komodo features.

# Supporting This Document And Author

The author is currently a Komodo Notary Node operator and Komodo Core Developer
who relies solely on cryptocoin income. If this document has helped you and
you would like to support future updates, you can send donations to
the KMD address RBSEv7nJ1wciriVyLFWotQ8tBvS2rKwYtz . Thanks!

# High Level Overview of DPoW

DPoW sends blockhash data from your coin, to KMD, which is then sent to Bitcoin,
providing a small coin with a small hashrate the security of BTC hashrate. This
data is embedded in OP\_RETURN metadata which Komodo Notary Nodes continuosly
send. Notary nodes run full nodes of Bitcoin, Komodo, and all Komodo Asset chains
and coins protected by KMD DPoW, and are the way data from one chain makes it onto
another chain.

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
the source code of Komodo has the public keys of notary nodes embedded in it's
source code. Only notary nodes have the corresponding private keys to make
transactions that the network will trust as valid notarizations. So DDoS
attacks can only stop notarization data from flowing temporarily, they can not
be used to knock actual notaries off-line and then impersonate or man-in-middle
the process.

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
safe to proceed with some action. Komodo provides an API to see if a txid of a
certain dpow-enabled coin has been notarized to KMD and BTC, so that exchanges
and other parties wanting to protect against double spends can make informed
decisions.

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
minute (the Komodo block interval) has a cost. There is a one-time yearly fee
of 300 KMD paid directly to jl777 to cover these costs. To protect against
future price increases of KMD (it's currently just over $1 USD), one may
purchase 6300 KMD, and then use the yearly 5% interest of 300 KMD to pay the
yearly notarization costs.

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

## Estimating Notarization Lag

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

It will be most likely be challenging to integrate a coin not derived from Bitcoin
but we are eager to see if it's possible. The currently available 0.11 and 0.15
header files can be used if the project is in C++ but if the coin is written
in another language then those header files must be ported to that language first.

## Adding some RPC methods

You will need to add a few RPC methods to your coin so that will allow
software to ask the node various questions about Merkle roots of Merkle roots
(MoMs). Crosschain proofs are Merkle roots of MoMs and are called MoMoMs.

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

