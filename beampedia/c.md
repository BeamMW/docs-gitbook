# C

## C++

A programming language developed by Bjarne Stroustrup at Bell Labs since 1979, as an extension of the C programming language. As a programming language it was designed with the following paradigms, among others:

- General-purpose: The opposite of a domain-specific programming languages, such as those developed for business applications, and thus contain built-in salary calculation functions. C++ can be used for the widest array of applications.

- Imperative: Uses statements that change a program's state. In much the same way that the imperative mood in natural languages expresses commands (e.g. “Add 2 plus 2”). Its opposite would be declarative programming, in which commands are conjured in the form of what the programmer wants, and leaves the software to find the route to do so (e.g. “sort a list”). Imperative languages assume a “very stupid, yet fast computer,” while declarative languages assume a “fairly competent” end which understand complex commands.

- Object-oriented: The foremost paradigm that distinguishes C++ from its predecessor, C. With object-oriented programming (OOP), programs are designed by making them out of “objects” that interact with one another. These can contain data and functions, and can be instantiated, their functions used, have their values modified, and destroyed (deleted from memory) during the lifetime of the program. These instances are based upon “classes,” which are their blueprints.

- Low-level memory manipulation: “Close to the metal programming,” such as direct operations on memory with pointers, in which data stored in the memory chip is accessed by its direct address. In other words, the language syntax can talk directly in CPU primitives and less like English. With this great power comes great responsibility, such as security (some memory addresses contain private information such as passwords), memory leaks (the program assumes memory is being used when it’s not), and insidious bugs. In other words, C++ allows you to write “unsafe code,” while other programming languages built fences around dangerous operations.  

- Compiled: Unlike scripting languages which need a host environment to run (such as JavaScript in a web browser), the program source text has to be processed by a compiler, producing hardware-specific instructions.

- Statically typed: The type of every entity (e.g. objects, values) must be known to the compiler at its point of use. The type of an object determines the operations that can be applied to it.

It was designed with a bias toward system programming and embedded, resource-constrained and large systems, with performance, efficiency and flexibility of use as its design highlights.

[Beam](/docs/beampedia/b#beam) uses C++17, the later iteration of C++.

## Confidential Assets

Mimblewimble can be extended to allows encoding multiple types of assets to be traded on the same blockchain. This idea has been described [here](https://blockstream.com/2017/04/03/blockstream-releases-elements-confidential-assets.html) by Andrew Poelstra. Enabling support for multiple asset types on Mimblewimble could be as simple as labeling each transaction output with a publicly visible “asset tag”; however, this would expose information about users’ financial behavior. Confidential Assets (CA) is a technology to support multiple asset types with blinding of asset tags, which builds on the privacy benefits of Confidential Transactions and extends the power and expressibility of blockchain transactions. As in Confidential Transactions, Confidential Assets allows anybody to cryptographically verify that a transaction is secure: the transaction is authorized by all required parties and no asset is unexpectedly created, destroyed, or transmuted. However, only the participants in the transaction are able to see the identity of asset types involved and in what amounts. There are two types of assets that can be implemented: predefined and custom. Each type has its advantages and limitations.

## Confidential Transactions

Originally proposed by Dr. Adam Back in 2013, in a Bitcointalk post title “bitcoins with homomorphic value.” Later implemented into a cryptographic library by Dr. Greg Maxwell (who expanded the concept) and Dr. Pieter Wuille from Blockstream.

Confidential Transactions hide amounts, substituting them for a cryptographic commitment in the form of a cryptographic hash. A commitment lets one keep a piece of data secret (the value in this case), but commit to it so that it cannot change it later. This commitment is a Pedersen Commitment, which allow to commit multiple values while preserving the addition property between them (called the homomorphic property).

This way the network can still verify that inputs and outputs add up –and thus no coins were created– without knowing the original values. It does this without adding any new basic cryptographic assumptions to the cryptocurrency system, with a manageable level of overhead, and without breaking the "pruning" property (discarding old values from the chain).

In a 2015 Bitcoin Knowledge Podcast interview, Dr. Adam Back gave the following illustration: “[with Confidential Transactions one can be] spending a tenth of a bitcoin to the coffee shop, and all the coffee shop learns is they receive a tenth of a bitcoin, but they don't know that the input was 100BTC and they don't know that the change is 99.9BTC, but they do know the encrypted 100BTC + encrypted 99.9BTC + 0.1BTC they received balances, so that they add up, together with the miners’ small fee.”

As a side-effect of its design, Confidential Transactions also enables the additional exchange of private "memo" data (such as invoice numbers or refund addresses) without any further increase in transaction size, by reclaiming most of the overhead of the CT cryptographic proofs.

## Consensus

In distributed computing, consensus is the overall system reliability in the presence of a number of faulty processes.

Consensus requires agreement among a number of processes or agents for a single data value. Some of these might fail or be unreliable in other ways, so consensus protocols must be fault-tolerant. The processes must put forth their candidate values, communicate with one another, and agree on a single value.

A consensus protocol tolerating halting failures must satisfy the following properties:

- Termination: A liveness property in which every correct (non-faulty) process eventually decides some value.
- Integrity: The ability to recover from the failure of a participating node.
- Agreement: A safety property, in which every correct process must agree on the same value.

Failures can refer to either a fail-stop, when a process stops communicating or a Byzantine fault, where a process send faulty or malicious data. The term Byzantine is taken from a simile by Leslie Lamport about a group of generals of the Byzantine army camped with their troops around an enemy city. Communicating only by messenger, the generals must agree upon a common battle plan. However, one or more of them may be traitors who will try to confuse the others. An algorithm resilient to such failures is called Byzantine fault-tolerant.

In the case of cryptocurrencies such as Bitcoin, nodes in the network must agree upon a common order of transactions. They do so by timestamping transactions into blocks, which are hashed and linked into a chain. Miners produce computational work into finding nonce which would produce a block hash under a certain size, called the target. If the block is valid under the protocol rules, nodes accept the block as canonical, reaching a consensus.

In the case that multiple blocks are valid, the longest chain –carrying the most proof-of-work– is accepted as canonical. This is the essence of Nakamoto consensus.

## Cryptocurrency

Protocols for transferring value, monetary or otherwise, electronically. Cryptocurrencies usually refer to systems that are anonymous. Such systems can be used to implement any quantity that is conserved, such as tokens, collectibles, assets, etc.

Their untraceability has made them attractive for illegal markets and money laundering, which was the reason these were their early adopters. Debates abound about the legality of alternative cash system to national currencies, as well as regulatory concerns over its anonymity aspects, the usefulness for evading taxes, and reporting requirements.

Privacy in an open society requires anonymous transaction systems. Until now, cash has been the primary such system. An anonymous transaction system is not a secret transaction system. An anonymous system empowers individuals to reveal their identity when desired and only when desired; this is the essence of privacy.

Although the cryptographical “digital cash” protocol was attempted in the past, such as in the work of David Chaum and projects such as B-money and Bitgold. It was with Bitcoin’s 2008 white paper and its implementation in the Bitcoin Core client that the concept took hold in the public imagination, which reverberated into unprecedented interest both from enthusiasts and speculators. Its speculative bubble in late-2017 was the largest in history, to the time of this writing.

The success of the cryptocurrency brought into public debate the very nature of money, the decoupling of government from currency, crypto-anarchy, libertarianism, Austrian economic principles, and the 2008’s economic crisis and its ensuing bank bailouts.

Beam takes a decade of cryptographic advances and trial and error to reimplement the protocol from scratch with Mimblewimble, Equihash proof-of-work, and other advances.

## Cryptographic Signature

In commercial practice, the validity of a contract is guaranteed by handwritten signatures. The essence of a signature is that only one person can produce it, but anybody can recognize. In a digital replacement, a user should be able to produce a message whose authenticity can be checked by anyone, by cannot be produced by anyone else.

A cryptographic signature of a message is code dependent on a secret is known only to the signer and on the content of the message being signed. A verifier can check the validity of the signature without revealing the secret.

Asymmetric encryption from public key cryptography provides a solution to the signature problem. Since in symmetric encryption, the two ends have the same key, so both the sender and receiver could have produced the encrypted message.

Encrypting a message only to prove authenticity is inefficient, a one-way hash function can map arbitrarily large data into smaller ones.

Digital signatures are a basic requirement for trust over the internet.