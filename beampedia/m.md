# M

## Merkle Tree

A cryptographic system, patented by Ralph Merkle from Stanford University in 1982, for providing digital signatures from multiple sources in an efficient way.

One time signatures, as embodied in the Diffie-Hellman algorithm are practical between a single pair of users who are willing to exchange a large amount of data necessary but they are not practical for most applications without further refinements.

The method is a “tree authentication” system, also called "binary hash tree," where the computation of the root forms a binary tree of recursive calls. Authenticating a particular leaf in the tree requires only those values starting from the leaf and progressing to the root.

A Merkle tree serves to encode blockchain data more efficiently and securely. The data structure is useful because it allows a client to verify a specific transaction without downloading the entire blockchain, only the relevant branch of the tree.

## MimbleWimble

A radical restructuring of the Bitcoin protocol, proposed by Tom Elvis Jedusor in 2016 in order to massively improve privacy and scalability of the digital currency. It is named after a curse in Harry Potter, since it aims “to prevent the blockchain from talking about all user's information.”

Tom Elvis Jedusor is the pseudonym of Voldemort in the French translation of Harry Potter. The protocol was described in a [document](/mimblewimble.txt) posted on the Bitcoin IRC channel, through TOR hidden services.

MimbleWimble extends the idea of confidential transactions and CoinJoin, which allows aggregation of transactions without requiring interactivity.

Blocks in Mimblewimble are composed of three lists: an input list, and output list, and an excesses list. The input list references the old outputs, and the output list contains confidential transaction outputs. The excesses list are lists of signatures and differences between outputs and inputs.

Mimblewimble does not support the use of Bitcoin scripting language, which makes it incompatible with the existing Bitcoin protocol.

## Mining

The term used to describe the process of proof-of-work computation in a cryptocurrency blockchain protocol to win the block reward. The primary purpose of mining is to produce a history of transactions, in a way that is computationally hard to modify the records. The mining reward is the mechanism for coin distribution in the system. Miners are also paid transaction fees, creating an incentive to secure the system.

Miners use various types of hardware. CPUs (central processing unit), due to their general purpose were used by the earliest cryptocurrency miners. GPU (graphics processing unit) produced a faster hashrate, and made CPU mining obsolete. The reason for these is that CPUs are architectured Arithmetic/Logic Units (ALUs), which are good in processing instructions, while GPUs are better at intensive, specialized repetitive tasks.

Also, FPGA (field-programmable gate array), integrated circuits programmed to operate hash functions, were used for mining until ASICs (application-specific integrated circuit), microchips specifically designed to process the hash function, vastly outperformed previous technologies.

Due to the unequal probabilistic distribution of mining rewards in most cryptocurrencies, “pools” formed to share the reward evenly according to hashrate contributed.

Some proof-of-work algorithms, such as Beam’s Equihash are memory intensive, so an ASIC is difficult to optimize for electric efficiency greater than over-the-counter GPUs.

## Moon

Earth’s sole natural satellite and the nearest celestial body. It was the first celestial object on which humans set foot. Its phases are a result of its orbit around the Earth and the varying angles of sunlight illuminating its surface. It is the primary cause of tides on Earth due to is gravitational pulls. Conversely, the gravitational forces of the Earth gradually slowed the Moon’s spin and “locked” on the side to face Earth.

Astronomers proposed several hypothesis for its formation 4.5 billion years ago, the most widely accepted explanation is that the Moon formed from the debris left over after a giant impact between Earth and a Mars-sized body called Theia.

The phrase “To The Moon!” has been used by inexperienced cryptocurrency traders in various internet social media in attempts to encourage a “pump and dump” on a recently purchased cryptocurrency (typically small, so-called “low cap” currencies, or more colloquially, “shitcoins”). That is, artificially inflate the price of an asset through false and misleading positive statements in order to sell the asset at a higher price.