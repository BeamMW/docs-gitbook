# B

## Beam

Beam is a scalable, privacy-oriented cryptocurrency based on the elegant breakthroughs of the Mimblewimble protocol, which completely conceal the values and metadata of transactions, in a prunable way which also reduces bloating on the blockchain. In addition to enhanced privacy and fungibility, this allows for much greater scalability.

None of the cryptographic protocols used in Beam require a trusted setup (unlike other privacy coins such as Zcash). Beam uses the Equihash’s Proof-of-Work algorithm, a memory-hard challenge which gives an early advantage to over-the-counter GPU miners. Like Bitcoin, scarcity is ensured by periodic halving. Complex transactions are possible, such as escrows, time-locked transactions, atomic swaps and more.

Beam has no premine, and won’t have an ICO. Backed by a treasury, emitted from every block during the first five years. Beam is being implemented from scratch by developers with years of experience in modern C++ system programming. Beam is an implementation of all the cryptographic breakthroughs and heuristics put forth by the cryptocurrency community over the years.

## Block

In computing, a block is a sequence of bytes or bits, usually containing records, and having a maximum length (block size). File systems and databases are based on blocks, which store and retrieve blocks of data.

The Bitcoin Whitepaper by Satoshi Nakamoto mentions the block in the context of its timestamping protocol. A block is a group of items to be timestamped and widely published with its hash. The timestamp proves that the data must have existed at the time in order to produce the hash. Each timestamp includes the previous block’s hash in its hash, thus forming a chain, with each additional timestamp reinforcing the ones before it.

Blocks have to be valid under the cryptocurrency protocol, nodes accept valid blocks by adding them to the chain of blocks, and work on extending them by [mining](/mining), and reject invalid blocks by ignoring them.

The block items are transactions, which are hashed in a Merkle Tree, with only the root included in the block’s hash. This proves the order of transactions under a cryptocurrency protocol to prevent double-spend attacks.

## Blockchain

A cryptographic solution to the double-spend problem without a trusted third party, proposed by Satoshi Nakamoto in “Bitcoin: A Peer-to-Peer Electronic Cash System”. He proposed that transactions should be publicly announced, and have a system for participants to agree on a single history of the order in which transactions were received. Mimblewimble removes the need for transactions to be publicly announced.

To achieve an order of transactions he defined a timestamp server. This works by taking the hash of a block of items (transactions in our case) to be timestamped and widely broadcast the hash. The timestamp proves that the data existed at the time in order to get the hash. Each timestamp includes the previous timestamp in its hash, forming a chain, with each additional timestamp reinforcing the ones before it.

The result is broadcasted over a peer-to-peer network with the help of a proof-of-work system, which requires computational effort to operate. Once the proof has been produced, the block cannot be changed without redoing the work. As later blocks are chained after it, the work to change the block would involve redoing all the blocks after it.

Proof-of-work also solves the problem of determining a majority decision making. Since if the vote on a valid block was based on a per IP-address, it could be subverted by a malicious actor allocating many IPs. The majority decision is represented by the longest chain, which has the greatest proof-of-work effort invested in it.

## Bulletproofs

Bulletproofs are short Proofs for Confidential Transactions. As a privacy solution in the blockchain, Dr. Maxwell proposed the use of Confidential Transactions, which hide amounts with the use of Pedersen Commitments. To verify that amounts are correct, range proofs are used to ensure that the sum of transaction inputs is greater than the sum of transaction outputs and that the amounts are not negative.

However, this is costly in terms of storage space, as range proofs need to be attached to every transaction.

Bulletproofs are short, non-interactive, zero-knowledge proofs that require no trusted setup. They are designed to enable efficient confidential transactions in Bitcoin and other cryptocurrencies.