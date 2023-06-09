> The following document is still under construction and is subject to changes

This page presents a list of terms and concepts used in Beam code

### Address

In Mimblewimble, unlike most other cryptocurrencies there are no addresses stored in the blockchian. However the Wallets still should be able to communicate to interactively create a transaction. Which is why in Beam, we provide the Secure Bulletin Board System (SBBS ) which allows wallets to send encrypted messages to each other via Beam Nodes. In SBBS=, each wallet is identified by one or more SBBS Addresses. Each Address is represented by a private/public key pair. All addreses are derived from the seed phrase, using a separate key chain, and are only stored in the wallet (not on the blockchain)

### Atomic Swap

Atomic Swap is an algorithm that allows to exchange value between two independent blockchains (for example Bitcoin and Beam) without having to rely on a centralized entity, such as exchange or any of the parties participating in the exchange. Atomic Swap is based on an ability to lock funds using Hash and Time Lock Contracts ([HTLC](https://en.bitcoin.it/wiki/Hash_Time_Locked_Contracts) ) on both chains. 

### Bulletproofs

Each UTXO, though confidential, should be a representation of a positive value. Creating negative coins could break the protocol and allow for arbitrary inflation. To prove that value is positive without revealing the value itself, Beam uses an implementation of the non interactive zero knowledge proof algorithm knows as [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf). Beam is using its own [implementation](https://github.com/BeamMW/beam/blob/master/core/ecc_bulletproof.cpp) of Bulletproofs.

### Coin

Internal representation of the single commitment for specific value in Beam Wallet. In reality, key id and value are stored and the actual blinding factors are generated each time. More detailed explanation on this structure is provided below

### Dandelion

An improvement of the basic P2P layer 'gossip' style protocol [whitepaper](https://arxiv.org/abs/1805.11060) making tracing of transaction source much more difficult through introduction of two phase transaction propagation. In 'Stem' phase, the transaction is sent through a series of random peers, during which each peer decides with a given probability whether to continue the 'Stem' or whether to 'Fluff'. In 'Fluff' phrase the transaction is broadcast to all known peers, effectively falling back to 'gossip' for transaction propagation.

### Elliptic Curve

Beam makes heavy use of [Elliptic Curve Cryptography](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography). Beam uses the [same curve as Bitcoin](https://en.bitcoin.it/wiki/Secp256k1) 

### Equihash

Beam uses Equihash Proof of Work mining algorithm. [description](https://en.wikipedia.org/wiki/Equihash). 

### Fly Client

In Beam implementation FlyClient protocol (located in project core) provides an abstraction for communication with either 'own' node and 'untrusted' node and hides implementation details from the Wallet. [ [implementation](https://github.com/BeamMW/beam/blob/master/core/fly_client.h) ]

### KIDF

Hierarchical key generator that allows to create new private and public keys from the master secret, generated from the seed_phrase and specific key index.

### Merkle Tree

[Merkle Tree](https://en.wikipedia.org/wiki/Merkle_tree) is a data structure representing a binary tree in which each non leaf node holds a cryptographic hash or the child nodes and which provides an efficient way to prove that a certain element is present in the tree in logarithmic time. In Beam, Merkle Tree is based on a [Radix tree](https://github.com/BeamMW/beam/blob/master/core/radixtree.h), in which the order of nodes is independent of the order in which the values were entered into the tree. Beam [implementation](https://github.com/BeamMW/beam/blob/master/core/merkle.h) uses generalized concept of Merkle Mountain Range, which can be seen as a collection fo Merkle Trees

### Own node

Beam wallet has two key modes of operation one with 'own' (trusted) node and one with 'untrusted' node. Regardless whether local or remote, the 'own' node knows the 'owner key' which is generated from the seed phrase and is used to tag specific UTXOs that belong to the wallet. In this case the node is considered trusted and the information received from it is treated accordindly. In case of the untrusted node, Beam Wallet employ the protocol called ChainWork (an implementation of the FlyClient idea by Benedict Bunz) which implements compact and reliable way of validation the information sent by the Node. 

### Peer

In the context of Node to Node communications, each node is a Peer for all others. Node stores and manages the list of Peers it is currently connected to and uses this list to propagate transactions and blocks. [Peer Manager](https://github.com/BeamMW/beam/blob/master/core/peer_manager.h) implements the logic of rating Peers and preferring those that are quicker to respond and provide valid information.

### SBBS

SBBS stands for Secure Bulletin Board System and provides an infrastructure for sending secure and encrypted messages between Beam Wallets to create a transaction. SBBS is part of the Beam Node. Each Address in the SBBS represents a private / public key pair used for encrypting the messages. Each wallet listens to a number of SBBS channels, derived from the Address to reduce the load on the wallet [ [protocol](https://github.com/BeamMW/beam/blob/master/core/proto.h) ]

### Seed Phrase 

This is the most secret key generated from the 12 word seed phrase. Using the seed phrase only it is always possible to reconstruct the entire UTXO set directly from the blockchain. In fact, this is the only information that can be extracted from the blockchain, all the other data including transaction history should be stored locally in the wallet.

### Transaction

In Mimblewimble transaction is created by both Sender and Receiver wallets and in its simplest form describes exchange of certain value between two (or more) participants. Unlike most other blockchains, transactions are actually meaningless in the context of the blockchain itself. They are however important in the context of the wallet. [ [implementation](https://github.com/BeamMW/beam/blob/master/wallet/base_transaction.h) ]