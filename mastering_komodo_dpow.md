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

DPoW sends Merkle hash data from your coin, to KMD, which is then sent to Bitcoin,
providing a small coin with a small hashrate the security of BTC hashrate. This
data is embedded in OP\_RETURN metadata which Komodo Notary Nodes continuosly
send. Notary nodes run full nodes of Bitcoin, Komodo, and all Komodo Asset chains
and coins protected by KMD DPoW, and are the way data from one chain makes it onto
another chain.

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
header file that matches that version of Bitcoin internals exactly. This was
first created by the author of Komodo, jl777, and then the author of this
document ported that header file to Bitcoin 0.11.x internals, which Zcash
and all Zcash forks have as their internals. So if your coin is a Zcash fork,
you should use the [komodo\_validation011.h](https://github.com/MyHush/hush/blob/e529ad8d35d4dabf66fa10982e056149414f179b/src/komodo_validation011.h)
header file. If your coin has
SegWit support, or is 0.14-0.16 internals, the [komodo\_validation015.h](https://github.com/jl777/chips3/blob/dev/src/komodo_validation015.h) file
is most likely the best starting point to integrate.

NOTE: The 0.11.x header file has some improvements which have yet to be ported
to the 0.15.x header file, specifically dynamic generation of Notary pubkey
address, which is currently hardcoded in that header.

For coins with older internals, such as BTC 0.10.x and earlier, use the BTC
0.11.x as a starting point, so less changes are required.

If your coin has made various internals changes and selectively added BIPs
or other internals changes, you will most likely need to make various changes
specific to your coin.

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

