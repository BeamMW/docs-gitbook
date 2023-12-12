# Beampedia

## A

### Account

Similar to what is commonly referred to as a [wallet](/docs/beampedia/w#wallet), an _account_ in [Beam](/docs/beampedia/b#beam) is a private account owned and operated by the user. 

An account contains all of the user's transaction accounting history, as well as the resulting current total _balance_.

A Beam account is [self-sovereign](https://www.ibm.com/blogs/blockchain/2018/06/self-sovereign-identity-why-blockchain/), which means that only the users can access and control their _balance_ and that the full responsibility rests on them. The account is usually stored in the user's computer.

This is achieved with _public key cryptography_, which has two separate keys: a public and a private key. The public key can be shared, and it can be used by the user to view the _balance_ without any security risks. The private key must be stored securely, and only used when signing a transaction.

### Address

An _address_ is an identifier that represents a possible destination for a payment. Addresses can be generated at no cost by any user. Like e-mail addresses, one can send a transaction to a person by sending to one of their addresses. However, unlike e-mail addresses, people may have different addresses and a unique address should be used for each transaction. 

A [Beam](/docs/beampedia/b#beam) _address_ is derived from a public key through the use of a cryptographic hash function. Addresses in Beam are longer than in most cryptocurrencies, as it uses _Confidential Transactions (CT)_, which need to include an additional public key in the address to encrypt a message to the receiver. This message includes information about the amount being transferred.

### Architecture

_Computer Architecture_ is the science and art of selecting and interconnecting software and hardware components to create computers that meet functional, performance and cost goals. 

A common problem in computer architecture is coping with complexity, much of it resulting from the need to increase the performance and scale of the computer system while meeting design goals and requirements.

The design goals of computer architecture are very often:

- Low cost
- Low energy
- Functionality  
- Reliability
- Performance

Technology is constantly changing, mostly due to _Moore’s Law_ which stipulates an exponential rate of transistor miniaturization. Often an improvement crosses a possibility threshold, which permits new applications that become its own self-sustaining market, such as smartphones.

Sometimes the improvement comes as a technological disruption, such as the transistor, microprocessors, multicore systems, and flash-based solid-state storage. Disruptions are necessary for scaling systems which can’t continue scaling forever. Software disruptions in the form of new algorithms, such as _public key cryptography_, enabled the scaling of cryptosystems for the internet age.

### Atomic Swaps

_Atomic swaps_ are the process of exchanging two different cryptocurrencies from separate blockchains in a trust-less and decentralized manner. This operation is based on scripts, and it allows users to trade their coins directly from their own personal crypto wallets. 

The biggest advantages of atomic swaps are in its decentralized nature. By removing the need for a centralized exchange or any other kind of mediator, cross-chain swaps can be executed by two (or more) parties without requiring them to trust each other. There is also an increased level of security because users retain full access and ownership of their funds unlike transacting through centralized exchange or third party. This form of peer to peer trading has much lower operational costs as trading fees are either very low or absent. Lastly, an enhanced level of anonymity is secured when atomic swapping privacy coins like Beam without an intermediary.

## B

### Beam

Beam is a scalable, privacy-oriented cryptocurrency based on the elegant breakthroughs of the Mimblewimble protocol, which completely conceal the values and metadata of transactions, in a prunable way which also reduces bloating on the blockchain. In addition to enhanced privacy and fungibility, this allows for much greater scalability.

None of the cryptographic protocols used in Beam require a trusted setup (unlike other privacy coins such as Zcash). Beam uses the Equihash’s Proof-of-Work algorithm, a memory-hard challenge which gives an early advantage to over-the-counter GPU miners. Like Bitcoin, scarcity is ensured by periodic halving. Complex transactions are possible, such as escrows, time-locked transactions, atomic swaps and more.

Beam has no premine, and won’t have an ICO. Backed by a treasury, emitted from every block during the first five years. Beam is being implemented from scratch by developers with years of experience in modern C++ system programming. Beam is an implementation of all the cryptographic breakthroughs and heuristics put forth by the cryptocurrency community over the years.

### Block

In computing, a block is a sequence of bytes or bits, usually containing records, and having a maximum length (block size). File systems and databases are based on blocks, which store and retrieve blocks of data.

The Bitcoin Whitepaper by Satoshi Nakamoto mentions the block in the context of its timestamping protocol. A block is a group of items to be timestamped and widely published with its hash. The timestamp proves that the data must have existed at the time in order to produce the hash. Each timestamp includes the previous block’s hash in its hash, thus forming a chain, with each additional timestamp reinforcing the ones before it.

Blocks have to be valid under the cryptocurrency protocol, nodes accept valid blocks by adding them to the chain of blocks, and work on extending them by [mining](/mining), and reject invalid blocks by ignoring them.

The block items are transactions, which are hashed in a Merkle Tree, with only the root included in the block’s hash. This proves the order of transactions under a cryptocurrency protocol to prevent double-spend attacks.

### Blockchain

A cryptographic solution to the double-spend problem without a trusted third party, proposed by Satoshi Nakamoto in “Bitcoin: A Peer-to-Peer Electronic Cash System”. He proposed that transactions should be publicly announced, and have a system for participants to agree on a single history of the order in which transactions were received. Mimblewimble removes the need for transactions to be publicly announced.

To achieve an order of transactions he defined a timestamp server. This works by taking the hash of a block of items (transactions in our case) to be timestamped and widely broadcast the hash. The timestamp proves that the data existed at the time in order to get the hash. Each timestamp includes the previous timestamp in its hash, forming a chain, with each additional timestamp reinforcing the ones before it.

The result is broadcasted over a peer-to-peer network with the help of a proof-of-work system, which requires computational effort to operate. Once the proof has been produced, the block cannot be changed without redoing the work. As later blocks are chained after it, the work to change the block would involve redoing all the blocks after it.

Proof-of-work also solves the problem of determining a majority decision making. Since if the vote on a valid block was based on a per IP-address, it could be subverted by a malicious actor allocating many IPs. The majority decision is represented by the longest chain, which has the greatest proof-of-work effort invested in it.

### Bulletproofs

Bulletproofs are short Proofs for Confidential Transactions. As a privacy solution in the blockchain, Dr. Maxwell proposed the use of Confidential Transactions, which hide amounts with the use of Pedersen Commitments. To verify that amounts are correct, range proofs are used to ensure that the sum of transaction inputs is greater than the sum of transaction outputs and that the amounts are not negative.

However, this is costly in terms of storage space, as range proofs need to be attached to every transaction.

Bulletproofs are short, non-interactive, zero-knowledge proofs that require no trusted setup. They are designed to enable efficient confidential transactions in Bitcoin and other cryptocurrencies.

## C

### C++

A programming language developed by Bjarne Stroustrup at Bell Labs since 1979, as an extension of the C programming language. As a programming language it was designed with the following paradigms, among others:

- General-purpose: The opposite of a domain-specific programming languages, such as those developed for business applications, and thus contain built-in salary calculation functions. C++ can be used for the widest array of applications.

- Imperative: Uses statements that change a program's state. In much the same way that the imperative mood in natural languages expresses commands (e.g. “Add 2 plus 2”). Its opposite would be declarative programming, in which commands are conjured in the form of what the programmer wants, and leaves the software to find the route to do so (e.g. “sort a list”). Imperative languages assume a “very stupid, yet fast computer,” while declarative languages assume a “fairly competent” end which understand complex commands.

- Object-oriented: The foremost paradigm that distinguishes C++ from its predecessor, C. With object-oriented programming (OOP), programs are designed by making them out of “objects” that interact with one another. These can contain data and functions, and can be instantiated, their functions used, have their values modified, and destroyed (deleted from memory) during the lifetime of the program. These instances are based upon “classes,” which are their blueprints.

- Low-level memory manipulation: “Close to the metal programming,” such as direct operations on memory with pointers, in which data stored in the memory chip is accessed by its direct address. In other words, the language syntax can talk directly in CPU primitives and less like English. With this great power comes great responsibility, such as security (some memory addresses contain private information such as passwords), memory leaks (the program assumes memory is being used when it’s not), and insidious bugs. In other words, C++ allows you to write “unsafe code,” while other programming languages built fences around dangerous operations.  

- Compiled: Unlike scripting languages which need a host environment to run (such as JavaScript in a web browser), the program source text has to be processed by a compiler, producing hardware-specific instructions.

- Statically typed: The type of every entity (e.g. objects, values) must be known to the compiler at its point of use. The type of an object determines the operations that can be applied to it.

It was designed with a bias toward system programming and embedded, resource-constrained and large systems, with performance, efficiency and flexibility of use as its design highlights.

[Beam](/docs/beampedia/b#beam) uses C++17, the later iteration of C++.

### Confidential Assets

Mimblewimble can be extended to allows encoding multiple types of assets to be traded on the same blockchain. This idea has been described [here](https://blockstream.com/2017/04/03/blockstream-releases-elements-confidential-assets.html) by Andrew Poelstra. Enabling support for multiple asset types on Mimblewimble could be as simple as labeling each transaction output with a publicly visible “asset tag”; however, this would expose information about users’ financial behavior. Confidential Assets (CA) is a technology to support multiple asset types with blinding of asset tags, which builds on the privacy benefits of Confidential Transactions and extends the power and expressibility of blockchain transactions. As in Confidential Transactions, Confidential Assets allows anybody to cryptographically verify that a transaction is secure: the transaction is authorized by all required parties and no asset is unexpectedly created, destroyed, or transmuted. However, only the participants in the transaction are able to see the identity of asset types involved and in what amounts. There are two types of assets that can be implemented: predefined and custom. Each type has its advantages and limitations.

### Confidential Transactions

Originally proposed by Dr. Adam Back in 2013, in a Bitcointalk post title “bitcoins with homomorphic value.” Later implemented into a cryptographic library by Dr. Greg Maxwell (who expanded the concept) and Dr. Pieter Wuille from Blockstream.

Confidential Transactions hide amounts, substituting them for a cryptographic commitment in the form of a cryptographic hash. A commitment lets one keep a piece of data secret (the value in this case), but commit to it so that it cannot change it later. This commitment is a Pedersen Commitment, which allow to commit multiple values while preserving the addition property between them (called the homomorphic property).

This way the network can still verify that inputs and outputs add up –and thus no coins were created– without knowing the original values. It does this without adding any new basic cryptographic assumptions to the cryptocurrency system, with a manageable level of overhead, and without breaking the "pruning" property (discarding old values from the chain).

In a 2015 Bitcoin Knowledge Podcast interview, Dr. Adam Back gave the following illustration: “[with Confidential Transactions one can be] spending a tenth of a bitcoin to the coffee shop, and all the coffee shop learns is they receive a tenth of a bitcoin, but they don't know that the input was 100BTC and they don't know that the change is 99.9BTC, but they do know the encrypted 100BTC + encrypted 99.9BTC + 0.1BTC they received balances, so that they add up, together with the miners’ small fee.”

As a side-effect of its design, Confidential Transactions also enables the additional exchange of private "memo" data (such as invoice numbers or refund addresses) without any further increase in transaction size, by reclaiming most of the overhead of the CT cryptographic proofs.

### Consensus

In distributed computing, consensus is the overall system reliability in the presence of a number of faulty processes.

Consensus requires agreement among a number of processes or agents for a single data value. Some of these might fail or be unreliable in other ways, so consensus protocols must be fault-tolerant. The processes must put forth their candidate values, communicate with one another, and agree on a single value.

A consensus protocol tolerating halting failures must satisfy the following properties:

- Termination: A liveness property in which every correct (non-faulty) process eventually decides some value.
- Integrity: The ability to recover from the failure of a participating node.
- Agreement: A safety property, in which every correct process must agree on the same value.

Failures can refer to either a fail-stop, when a process stops communicating or a Byzantine fault, where a process send faulty or malicious data. The term Byzantine is taken from a simile by Leslie Lamport about a group of generals of the Byzantine army camped with their troops around an enemy city. Communicating only by messenger, the generals must agree upon a common battle plan. However, one or more of them may be traitors who will try to confuse the others. An algorithm resilient to such failures is called Byzantine fault-tolerant.

In the case of cryptocurrencies such as Bitcoin, nodes in the network must agree upon a common order of transactions. They do so by timestamping transactions into blocks, which are hashed and linked into a chain. Miners produce computational work into finding nonce which would produce a block hash under a certain size, called the target. If the block is valid under the protocol rules, nodes accept the block as canonical, reaching a consensus.

In the case that multiple blocks are valid, the longest chain –carrying the most proof-of-work– is accepted as canonical. This is the essence of Nakamoto consensus.

### Cryptocurrency

Protocols for transferring value, monetary or otherwise, electronically. Cryptocurrencies usually refer to systems that are anonymous. Such systems can be used to implement any quantity that is conserved, such as tokens, collectibles, assets, etc.

Their untraceability has made them attractive for illegal markets and money laundering, which was the reason these were their early adopters. Debates abound about the legality of alternative cash system to national currencies, as well as regulatory concerns over its anonymity aspects, the usefulness for evading taxes, and reporting requirements.

Privacy in an open society requires anonymous transaction systems. Until now, cash has been the primary such system. An anonymous transaction system is not a secret transaction system. An anonymous system empowers individuals to reveal their identity when desired and only when desired; this is the essence of privacy.

Although the cryptographical “digital cash” protocol was attempted in the past, such as in the work of David Chaum and projects such as B-money and Bitgold. It was with Bitcoin’s 2008 white paper and its implementation in the Bitcoin Core client that the concept took hold in the public imagination, which reverberated into unprecedented interest both from enthusiasts and speculators. Its speculative bubble in late-2017 was the largest in history, to the time of this writing.

The success of the cryptocurrency brought into public debate the very nature of money, the decoupling of government from currency, crypto-anarchy, libertarianism, Austrian economic principles, and the 2008’s economic crisis and its ensuing bank bailouts.

Beam takes a decade of cryptographic advances and trial and error to reimplement the protocol from scratch with Mimblewimble, Equihash proof-of-work, and other advances.

### Cryptographic Signature

In commercial practice, the validity of a contract is guaranteed by handwritten signatures. The essence of a signature is that only one person can produce it, but anybody can recognize. In a digital replacement, a user should be able to produce a message whose authenticity can be checked by anyone, by cannot be produced by anyone else.

A cryptographic signature of a message is code dependent on a secret is known only to the signer and on the content of the message being signed. A verifier can check the validity of the signature without revealing the secret.

Asymmetric encryption from public key cryptography provides a solution to the signature problem. Since in symmetric encryption, the two ends have the same key, so both the sender and receiver could have produced the encrypted message.

Encrypting a message only to prove authenticity is inefficient, a one-way hash function can map arbitrarily large data into smaller ones.

Digital signatures are a basic requirement for trust over the internet.

## D

### Dandelion

A privacy enhancing transaction routing mechanism, proposed by [Giulia Fanti](https://www.andrew.cmu.edu/user/gfanti/) et al. based on Bitcoin’s BIP-156, that provides anonymity guarantees against deanonymization attacks.

Without it, nodes transmit the transaction to their peers, a process known as diffusion, in a statistically symmetric way. This allows surveillance mechanisms to triangulate the IP location of the transaction, by strategically placing colluding spying nodes around routing locations.

Dandelion mitigates this by sending transaction over a random path before diffusion, in the same way, that dandelion stems fly through the air before spreading its seeds. Transactions travel silently along a path during an anonymous “stem phase,” and are then diffused to the network during the “fluff phase.”

While other privacy solutions aim to protect individual users, Dandelion protects anonymity by limiting the capability of adversaries to deanonymize the entire network.

### Data Directory

The disk location where Beam’s data files are stored, including the wallet data file.

Wallet files will be created in following folders:

- Mac: `/Users/{your_user_name}/Library/Application Support/Beam Wallet/`
- Windows: `\Users\{your_user_name}\AppData\Local\Beam Wallet`
- Linux: `/home/{your_user_name}/.local/share/Beam Wallet`

Upon running a Beam node, log files are located in the logs folder, which resides in the node folder. Once started, the node will create a node.db file in the same folder it is located. This file will store an internal state of the node. Upon the first launch, the node will download current blockchain history in batch mode as a single large macroblock. After the initial sync is complete, the node will continue to sync blocks and individual transactions from the current blockchain Tip (height) and onwards.

## E

### Elliptic Curve Cryptography

An elliptic curve is the set of points that satisfy an equation in the form of y^2 = x^3 + ax + b. It has interesting properties, such as symmetry over the x-axis. The second property is that any two points on the curve would make a line which crosses a third point on that curve. If the two points are exactly vertical, the third point is considered to be at infinity. If the two points are the same, the derivative of the curve is taken at that point (the slope of curve at that point). Entering two points and receiving the third one is defined to be the addition of two points. The third point is later reflected over the x-axis.

The idea behind encryption with an elliptic curve is of taking a plain message and defining it as a point in the curve (since a message can be turned into a long number). This point is added to itself over and over again a k number of times. The message turned into a point, and the point turned into another point in the curve multiple times, like a pool ball bouncing around. After some other manipulations depending on the system, we have our cypher-text, i.e., our encrypted message. Without knowing k (the private key), it is prohibitively hard to decrypt the message. i.e., without knowing how many times a pool ball bounced, it is impossible to know where it started.

In practice, the curve be made of discrete points, and not a continuous line, since only integer numbers are used. Furthermore, if we limit ourselves to a subset of numbers by picking a prime number as a maximum, and modulate through it so as numbers start over from 0 if they surpass the maximum, we get a scattered (but symmetrical) array of dots.

An elliptic curve system can be configured by picking:

- A prime number as a maximum,
- A curve equation,
- A public point on the curve.

The private number is the number of times the public point will be added to itself. Computing the private key from the public key is called the elliptic curve discrete logarithm function, and is a trapdoor function.

Elliptic curve cryptography is a modern system and very efficient in time and space. It is slated to replace the ubiquitous RSA system based on prime factorization. Unlike factoring, there doesn’t appear to be shortcuts, and mathematicians still haven’t found an algorithm to solve this problem better than a naive approach. As such, its security is exponentially stronger than RSA.

It has been implemented in by the U.S. government to encrypt internal communications, the Tor project uses it, and it is the new default method for authentication for secure web browsing over SSL/TLS. Among cryptocurrencies, it is used by Bitcoin of course, as well as Ethereum, EOS and Zcash. And as the wiki page says, the list is incomplete, and we can help by expanding it.

Beam uses the libsecp256k1 optimized C library implemented in the Bitcoin Core client, for operation on a curve of type secp256k1, see also this answer. Secp256k1 refers to the parameters of the elliptic curve, as defined in Standards for Efficient Cryptography (SEC). The resulting curve is that of y^2 = x^3 + 7, defined over the field Z (integer numbers) modulo 2^256-2^32-977. Be prepared to use huge integer arithmetic, since this number is several lines long.

### Emission

The act of issuing new coins, minting. In Beam, coins are emitted smoothly, as the reward changes with each new block. This allows a predictable steady growth of money supply determined by the block reward.

When a block is discovered, the discoverer may award themselves a certain number of coins, which is agreed-upon by everyone in the network. This is in order to incentivize mining activity, and for the miners to have a stake in the system to remain honest.

Beam has set in its protocol a periodic halving of reward, with a total amount of approximately 250 million coins. As the number of new coins that are allowed to create in each block dwindles, mining fees will make up a much more important percentage of a miner’s income.

### Encryption

Methods of deliberately distorting a message so that the legitimate receiver can recover it easily while being nearly impossible for an eavesdropper to do the same.

In symmetric encryption, a personal encryption key cipher known only to the sender and its intended receiver is used to control the encryption of the data. Manual encryption has been used since Roman times (such as Caesar cipher, used by Julius Caesar in his private correspondence). Mechanical systems improved on this, and were used during the world wars. With digital hardware, the most commonly used algorithm is Data Encryption Standard (DES), now phased out for Advanced Encryption Standard (AES).

These schemes allow for cryptographic communications among people who have made prior preparation for cryptographic security. However, it is unviable for large telecommunications systems, where users would have to wait for a key to be sent over secure, physical means.

Asymmetric encryption, or public-key cryptography, requires a pair of keys: one for encryption and one for decryption. Public-key cryptography is a modern breakthrough designed by Whitfield Diffie and Martin Hellman, two academics who disrupted the government monopoly on encryption. It uses trapdoor one-way functions so that encryption may be done by anyone with access to the "public key" but decryption may be done only by the holder of the "private key." A public key may be freely published in directories, while the corresponding private key is closely guarded.

The RSA algorithm (developed by Rivest, Shamir, and Adleman) is the most widely used form of public key encryption. Pretty Good Privacy (PGP) was written by Phillip Zimmerman as an open implementation of RSA which empowered people to take their privacy into their own hands. As EFF co-founder John Perry Barlow put it: "You can have my encryption algorithm... when you pry my cold dead fingers from my private key."

For performance reasons, a popular scheme is to use RSA to transmit session keys and then a high-speed cipher like DES for the actual message text. Public-key cryptography enabled private communication and financial transactions over the internet.

Elliptic Curve Cryptography (ECC) is a modern system and very efficient in computational time and space. It is slated to replace RSA. Based on the trap-door properties of points on elliptic curves. Unlike prime factoring, there doesn’t appear to be any mathematical shortcuts. As such, its security is exponentially stronger than RSA.

### Equihash

Equihash is a mining algorithm proposed by Alex Biryukov and Dmitry Khovratovich as part of the University of Luxembourg research group CryptoLUX. It proposed a fast, memory-asymmetric and optimization/amortization-free PoW, based on the generalized birthday problem. In plain English, the problem states that: “Given a year with d days, what is the n amount of randomly chosen people to have a birthday coincidence of at least 50%?” Which can be generalized into k-types of people, finding common birthdays between them. The generalization states that: “Given k lists of n-bit values, find some way to choose one element from each list so that the resulting k values XOR to zero.”

Zcash Foundation implemented Equihash PoW, specified with parameters of n=200, k=9, which dictate how much memory space/bandwidth will be needed to find a solution. In June 2018, Bitmain released the ASIC Antminer Z9 mini, which managed to mine Equihash much more efficiently than commodity CPUs and GPUs by interfacing with SRAM chips at relatively low cost. Zcash founder Zooko Wilcox wrote in a forum post that he never committed to ASIC-resistance since it would be impossible in the long-term.

Equihash, however, allows for an initial widespread distribution of the coins and mining. Beam will set the parameters giving CPU and GPU miners significant head start over ASICs. Thus we will achieve a balance between the tradeoffs of a widespread, democratic coin distribution and the inevitable but synergic attack-resistance from ASIC miners.

## F

### Financing

The process of providing funds for business activities, making purchases or investing. In traditional financing, there are two main types of funding: debt and equity. Debt is a loan that must be paid back, often with interest. Equity gives ownership stakes in the company to the shareholder. Financing takes advantage of the fact that some have a surplus of money that they wish to put to work, while others demand money to undertake investment.

Cryptocurrency has provided a new venue for financing, as blockchain-based ventures raise capital by selling cryptographically secure digital assets (tokens). This can be in the form of a general-purpose medium of exchange and store-of-value cryptocurrency, a token which represents conventional security (security token), or a token which represent a right to a service provided in the network (utility token).

Utility tokens leverage the value of the network and the market demand of the token itself. This decouples the ownership of tokens from equity in the venture, and (theoretically) puts the incentive in the long-term development of the platform. However, utility tokens have found very short-term success, and their regulatory uncertainty was a breeding ground for frauds. The market has thus shifted to slower, regulated security tokens offerings.

Financing in Beam can be achieved with Confidential Assets, which allow the issuance of custom assets, such as tokens. This not only preserves the privacy of the issuer, but also the privacy of the issued token which is known only to the involved parties.

### FOMO

Acronym for “fear of missing out.” The social anxiety over missing out on something important. The fear is tied to regret and imagined possibilities, exacerbated by social media.
This is a subject for behavioral economics, which studies the effects of psychological, cognitive, emotional, cultural and social factors on the economic decisions of individuals and institutions. Behavioral economics often contradict the classical economic theory, which developed under the assumption that human beings are perfectly rational agents.

Dan Ariely, Professor of behavioral economics at Duke University, explained that FOMO is caused by anticipation of regret, as we compare our present state on what “we could have been.” This is called “reference dependence,” as defined by Daniel Kahneman, (winner of the 2002 Nobel Prize in economics) in his 1979 book Prospect Theory: An Analysis of Decision Under Risk.

### Fungibility

In economics, fungibility is the property of a good or a commodity whose individual units are essentially interchangeable, as fungibility implies equal value between the assets. Fungibility refers only to the equivalence of each unit of a commodity with other units of the same commodity and not to the exchange of one commodity for another, which is barter.

Currency is fungible: one banknote is interchangeable with any other banknote like it. Gold of the same grade and weight is fungible. On the other hand diamonds are considered unique and are not perfectly fungible, variations make it difficult to find several diamonds expected to have the same value. Fungibility does not imply liquidity, and vice versa. Diamonds trading is liquid, yet individual diamonds are not interchangeable.

In cryptocurrencies, privacy is essentially tied to fungibility. Cryptocurrencies with a public ledger such as Bitcoin often lose fungibility, as some coins are treated as more acceptable than others. Coins involved in questionable transactions can taint entire wallets since values are merged. Tainted coins can be accepted without knowledge, or can even be sent into an unknowing account, resulting in a loss of reputation.

## G

### Groth

A Groth is the smallest unit of Beam recorded on the blockchain. It is a one hundred millionth of a single Beam (0.00000001 BEAM).

The unit has been named in homage to [Jens Groth, Prof. of Cryptology at UCL](http://www0.cs.ucl.ac.uk/staff/j.groth/). Learn more about how Prof. Groth became a Fractional monetary unit [here](https://www.benthamsgaze.org/2019/05/22/efficient-cryptographic-arguments-and-proofs-or-how-i-became-a-fractional-monetary-unit/).

## H

### Hash

One way hash functions, also called fingerprints or cryptographically secure checksums. A one-way function is one which is easy to compute but difficult (effectively impossible) to invert. The hash function maps data of arbitrary size into a string of fixed size, using a one-way compression function.

Ideally, a cryptographic hash function should be:

- deterministic, the same message always results in the same hash
- quick to compute
- infeasible to generate a message from its hash
- a small change to a message should change the hash value completely, so no correlations could be inferred
- infeasible to find two different messages with the same hash value (collision)

The most common hash algorithms are MD5 (message-digest 5), which has been deprecated for SHA-1 (Secure Hash Algorithm), also phased out to SHA-2 and SHA-3. The algorithms above use the Merkle–Damgård construction, while SHA-3 uses Keccak ([sponge construction](https://en.wikipedia.org/wiki/Sponge_function)). The newer BLAKE algorithm is based on ChaCha stream cipher. BLAKE is a faster algorithm when run on 64-bit and ARM architectures, used in the Equihash proof-of-work algorithm, and as a key derivation function.

### Hold

A passive investment strategy, also called “buy and hold,” for which the investor buys a cryptocurrency asset and holds it for a long period of time regardless of fluctuations in the market. In the case of high volatility markets such as cryptocurrency.

The first use of the term dates back to December 2013, in a bitcointalk post titled “[I AM HODLING](https://bitcointalk.org/index.php?topic=375643.msg4022997#msg4022997)” by Sr. Member GameKyuubi after (in his own words) had some whiskey.

GameKyuubi’s reasoning was that in extremely volatile markets, at a zero-sum game such as trading, inexperienced traders often lose to more seasoned investors and should focus instead on long term investments. In his own words: “You only sell in a bear market if you are a good day trader or an illusioned noob. The people in between (sic) hold. In a zero-sum game such as this, traders can only take your money if you sell.”

In the case of pump and dump schemes, often seen in cryptocurrencies of small market cap, the advice can be malicious, as investors attempt to exit the market before those who are piously holding an asset which is rapidly becoming worthless.

The aim of cryptocurrencies is to become a self-sovereign store of value, not necessarily a financial investment. Many cryptocurrency evangelists hope for their currencies to appreciate due to the devaluation of fiat government-backed currencies into hyperinflation, eventually transacting exclusively in cryptocurrency. Holding advice is thus never to exchange cryptocurrency into fiat-based currencies, and never spending cryptocurrencies until their ultimate global adoption.

## L

### Lambo

Abbreviation of Lamborghini, an Italian brand and manufacturer of luxury sports cars, SUVs, and tractors. The cars are some of the most sought-after sports cars in the world, is among the fastest and most expensive. The abbreviation seldom includes Lamborghini’s tractor line, noted for their high-performance hydraulic pump system and customizable transmission configuration.

Due to the consonant digraph in the name (“gh”), unfamiliar to non-Italian speakers, the abbreviation has spared many of typing errors, universally accepted as the most revealing sign of low IQ on internet social media, even in informal conversations.

It is often used as a comparative measurement of value decoupled from national currencies. At the 2017’s all-time high (ATH), one bitcoin was worth 1/20th of a Lamborghini Aventador. It is helpful for comprehension as a desirable, tangible asset, unlike wealth “on paper” when stated in terms of fiat currency.

The simile illustrates the rapid value increase of cryptocurrencies since on May 2010, Laszlo Hanyecz paid a Bitcoin Talk user 10,000 BTC for two Papa John’s pizzas, regarded as the first tangible purchase with a cryptocurrency.

“When Lambo?” is the interrogative equivalent to the phrase “To the Moon!” in internet social media. Sometimes combined as “Lambos on The Moon,” not accounting for shipping expenses, Lamborghinis’ low ground clearance ineffective on cratered surfaces, stiff suspension for low gravity, or the absence of atmospheric oxygen required for the operation of a combustion engine.

Purchasing a high-end sports car with cryptocurrency revenues should proceed after careful compliance with tax regulation, as government agencies are prone to inquire about luxurious concessions, particularly if someone is unemployed.

### Lelantus

Beam has implemented several improvements to the original Mimblewimble protocol to obfuscate the transaction graph. A hybrid of MW and Lelantus is bring introduced now, which should be a huge step forward in this direction.

Disclaimer:
The Lelantus protocol is the work of Zcoin's cryptographer Aram Jivanyan as part of their research to improve their privacy protocol. To fit our needs and utilize the full power of MW we made several modifications to the original protocol.

Lelantus uses standard cryptographic assumptions and allows the creation of a shielded UTXOs pool with strong anonymity. It also does not require any trusted setup. Lelantus-MW will dramatically increase the UTXO anonymity set and make it virtually impossible to establish links between different UTXOs. On top of that, Lelantus transactions will provide one-sided payments which allows for sending and receiving Beam without Mimblewimble’s interactivity requirement.

Read the full explanation on Lelantus [here](https://github.com/BeamMW/beam/wiki/Lelantus-MW).

## M

### Merkle Tree

A cryptographic system, patented by Ralph Merkle from Stanford University in 1982, for providing digital signatures from multiple sources in an efficient way.

One time signatures, as embodied in the Diffie-Hellman algorithm are practical between a single pair of users who are willing to exchange a large amount of data necessary but they are not practical for most applications without further refinements.

The method is a “tree authentication” system, also called "binary hash tree," where the computation of the root forms a binary tree of recursive calls. Authenticating a particular leaf in the tree requires only those values starting from the leaf and progressing to the root.

A Merkle tree serves to encode blockchain data more efficiently and securely. The data structure is useful because it allows a client to verify a specific transaction without downloading the entire blockchain, only the relevant branch of the tree.

### MimbleWimble

A radical restructuring of the Bitcoin protocol, proposed by Tom Elvis Jedusor in 2016 in order to massively improve privacy and scalability of the digital currency. It is named after a curse in Harry Potter, since it aims “to prevent the blockchain from talking about all user's information.”

Tom Elvis Jedusor is the pseudonym of Voldemort in the French translation of Harry Potter. The protocol was described in a [document](/mimblewimble.txt) posted on the Bitcoin IRC channel, through TOR hidden services.

MimbleWimble extends the idea of confidential transactions and CoinJoin, which allows aggregation of transactions without requiring interactivity.

Blocks in Mimblewimble are composed of three lists: an input list, and output list, and an excesses list. The input list references the old outputs, and the output list contains confidential transaction outputs. The excesses list are lists of signatures and differences between outputs and inputs.

Mimblewimble does not support the use of Bitcoin scripting language, which makes it incompatible with the existing Bitcoin protocol.

### Mining

The term used to describe the process of proof-of-work computation in a cryptocurrency blockchain protocol to win the block reward. The primary purpose of mining is to produce a history of transactions, in a way that is computationally hard to modify the records. The mining reward is the mechanism for coin distribution in the system. Miners are also paid transaction fees, creating an incentive to secure the system.

Miners use various types of hardware. CPUs (central processing unit), due to their general purpose were used by the earliest cryptocurrency miners. GPU (graphics processing unit) produced a faster hashrate, and made CPU mining obsolete. The reason for these is that CPUs are architectured Arithmetic/Logic Units (ALUs), which are good in processing instructions, while GPUs are better at intensive, specialized repetitive tasks.

Also, FPGA (field-programmable gate array), integrated circuits programmed to operate hash functions, were used for mining until ASICs (application-specific integrated circuit), microchips specifically designed to process the hash function, vastly outperformed previous technologies.

Due to the unequal probabilistic distribution of mining rewards in most cryptocurrencies, “pools” formed to share the reward evenly according to hashrate contributed.

Some proof-of-work algorithms, such as Beam’s Equihash are memory intensive, so an ASIC is difficult to optimize for electric efficiency greater than over-the-counter GPUs.

### Moon

Earth’s sole natural satellite and the nearest celestial body. It was the first celestial object on which humans set foot. Its phases are a result of its orbit around the Earth and the varying angles of sunlight illuminating its surface. It is the primary cause of tides on Earth due to is gravitational pulls. Conversely, the gravitational forces of the Earth gradually slowed the Moon’s spin and “locked” on the side to face Earth.

Astronomers proposed several hypothesis for its formation 4.5 billion years ago, the most widely accepted explanation is that the Moon formed from the debris left over after a giant impact between Earth and a Mars-sized body called Theia.

The phrase “To The Moon!” has been used by inexperienced cryptocurrency traders in various internet social media in attempts to encourage a “pump and dump” on a recently purchased cryptocurrency (typically small, so-called “low cap” currencies, or more colloquially, “shitcoins”). That is, artificially inflate the price of an asset through false and misleading positive statements in order to sell the asset at a higher price.

## N

### Node

In the mathematical branch of graph theory, a node (or vertex, from Latin nodus, ‘knot’), is the fundamental unit of which graphs are formed. A graph consists of a set of nodes and a set of edges connecting them.

In computer science, nodes are devices on a network or data points on a data structure. On the Internet Protocol specifically, a node is any device with an IP address, which serves as identification and for location addressing.

In distributed systems, nodes are networked computers, which have the same goal for their work without a shared clock. They might have shared memory or operate under their own private memory.

A cryptocurrency network is a distributed network that reaches consensus on shared memory, the distributed ledger or blockchain. Any other device connected to the network is a node, such as lightweight wallets. The responsibilities of a “full node” are in fully validating transactions and blocks, enforcing the consensus rules. Most nodes also accept transactions and blocks from other full nodes, validating them, and relaying them further to other nodes.

Miners, which might or might not be required to be full nodes in the network, depending on the cryptocurrency protocol. In most implementations, including Bitcoin, miners only need to know about the prior block, while full nodes require the entire blockchain.

Nodes work all at once with little coordination. They do not need to be identified since messages are not routed to any particular place and only need to be delivered on a best effort basis. Nodes can leave and rejoin the network at will, accepting the proof-of-work chain as proof of what happened while they were gone.

## O

### Offline Transactions

In Mimblewimble there are no addresses, and transactions need to be built interactively by the participating parties. This requires that both parties set direct socket connection for every transaction, which can be impractical, and doesn’t allow for transactions while the receiving party is offline.

A Secure Bulletin Board (SBBS) system solves this problem, enabling offline transactions, which makes the user experience the same as other cryptocurrencies such as Bitcoin. With SBBS, client wallets can exchange messages by store-and-forward nodes.

With SBBS, messages are private when addressed to an individual, such messages are encrypted with a public key before they are sent, without having to trust relaying nodes. Other messages can be set to public and be unencrypted, or sent to specific channels.

### Opt-in Auditability

Auditability allow business to manage in a confidential way while being fully compliant.

The Auditability feature is an extension that we added to the original Mimblewimble protocol, allowing businesses or private individuals to report their financial history to their auditors or any other party of their choosing in a secure and provable way. If the user chooses to use Auditability, his/her wallet generates a key pair for audit.

With Auditability, the user can include additional information (e.g. info about the sender/receiver, invoice details etc) thus making it easier for the auditor to classify each transaction.
The auditor is not, able to create any transaction, nor spend any funds. The auditor can retrieve all transactions form the wallet and the blockchain, make sure that the transactions comply with the presented documents. Auditability is strictly an opt-in feature. If the user hasn’t chosen to be auditable, there is no way anyone can retrieve any information about his transactions from the blockchain.

## P

### Pedersen Commitment

The basic tool that Confidential Transactions is based on. A commitment scheme lets you keep a piece of data secret but commit to it so that you cannot change it later. Two properties need to be satisfied: binding and hiding. Binding makes sure that commitments cannot be changed later, while hiding ensures that adversaries are unable to find the original value.

A simple commitment scheme can be constructed using a cryptographic hash, by hashing together the data and a blinding factor, and revealing the hash:

*commitment = SHA256( blinding_factor || data )*

This hash is the commitment, and the operation cannot be reversed to reveal the original data. The data can be later revealed together with the blinding factor and can be verified by hashing them with the same hashing algorithm.

The blinding factor is present because, without one, the data could be potentially guessed, –if the data is small and simple– by comparing the guess to the commitment.

A Pedersen commitment works like the above but with an additional property: commitments can be added. The sum of a set of commitments is the same as a commitment to the sum of the data, with a blinding key set as the sum of the blinding keys.

*C(BF1, data1) + C(BF2, data2) == C(BF1 + BF2, data1 + data2)*

### Privacy

Privacy is the prevention of unauthorized extraction of information from communications over an insecure channel, it is the best-known problem that field of cryptography attempts to solve.

There are many reasons why people need privacy: personal, psychological, financial, social, etc. People encrypt for the same reason they close and lock their doors. Encryption can protect criminals, but so do locked doors.

Although there is no explicit "right of privacy" enumerated in the U.S. Constitution, the assumption that an individual is to be secure from absent a valid warrant is central, and dates back to the Carta Magna. That includes random searches and invasive tactics to catch criminals.

There has never been a ruling or law that persons have to speak in a language understandable by wiretappers. Similarly, there hasn’t been a rule banning encrypted communication.

Privacy is essentially tied to fungibility in cryptocurrencies. As cryptocurrencies with a public ledger such as Bitcoin often lose fungibility, as some coins are treated as more acceptable than others. A user can unknowingly accept coins involved in questionable transactions, which can taint the entire amount of his/her wallet and result in a loss of reputation.

Beam achieves privacy by default, unlike other cryptosystems which require an opt-in. Zcash or Monero, for instance, requires users to choose a private transaction, an option which significantly slows down the transaction, and consequently is only used by 4% of the network, in the case of Zcash.

## S

### SBBS

In Mimblewimble there are no addresses, and transactions need to be built interactively by the participating parties. This requires that both parties set direct socket connection for every transaction, which can be impractical, and doesn’t allow offline transactions.

A Secure Bulletin Board (SBBS) system solves this problem, enabling offline transactions, which makes the user experience the same as other cryptocurrencies such as Bitcoin. With SBBS, client wallets can exchange messages by store-and-forward nodes. The messages are encrypted with public key cryptography, over elliptic curve cryptography (ECC) algorithms.

Originally, a bulletin board system is a computer server running software that allows users to connect with a client. Once logged, the user can upload or download messages through public message boards or private email, as well as file downloads (often text files), and if the client allowed, direct chatting and even text-based multiplayer games. This is the ecosystem where public key cryptography, such as Phil Zimmerman’s PGP implementation originally spread.

The architecture of a BBS is replicated with Beam’s nodes and wallets, as described in the [GitHub repository](https://github.com/BeammW/beam/wiki/Secure-bulletin-board-system-(SBBS)), and a [Medium article](https://medium.com/beam-mw/the-secure-bulletin-board-system-sbbs-implementation-in-beam-a01b91c0e919). With SBBS, messages are private when addressed to an individual, such messages are encrypted with a public key before they are sent, without having to trust relaying nodes. Other messages can be set to public and be unencrypted or sent to specific channels.

### Scriptless Scripts

Scriptless Script technology allows the blockchain to implement a variety of transaction types. This technology includes support for atomic swapping, escrow, and time-locked transactions and more.

Atomic swapping enables users to exchange cryptocurrencies without a middle-man or exchange. Furthermore, atomic swapping is faster and cheaper than centralized alternatives.

In addition to atomic swapping, this technology allows escrow transactions. Escrows are commonplace in over-the-counter (OTC) cryptocurrency transactions. Escrows allow the buyer and seller to verify funds and thus, a secure transaction. Using this technology buyers and sellers can initiate escrow transactions safely and reliably.

Time-locked transactions are exactly what they sound like: transactions that will not go through until a time condition is met. Cryptocurrency based time-locked transactions have merit as no single party can alter terms, one of the benefits of decentralization.

## W

### Wallet

A wallet is a container used to manage the public and private keys in a public key cryptosystem. Beam, as all other cryptocurrencies use public keys to sign transactions, which prove that it is the user who agreed to pay an amount to an address. The signature, along with the corresponding signed data, is used to update the ledger, and write the updated amounts to the corresponding accounts, including fees.

The wallet itself does hold any coins or even have a balance. It is the software implementation which displays this information, retrieving the information from an updated node.

Wallets can be nondeterministic, where each key is generated from a random number, or deterministic, where the keys are derived from a single master key (the seed). The most popular type is the HD wallet (hierarchical deterministic wallet). These seeds are often encoded as the mnemonic code works, for better user experience and security.

Beam Wallet files will be created in following folders:

- Mac: `/Users/{your_user_name}/Library/Application Support/Beam Wallet/`
- Windows: `\Users\{your_user_name}\AppData\Local\Beam Wallet`
- Linux: `/home/{your_user_name}/.local/share/Beam Wallet`

## Z

### Zero-Knowledge Proof (ZK)

Proofs in which no knowledge of the actual proof is conveyed. Alice proves to Bob that she is indeed in possession of some piece of knowledge without revealing any of that knowledge. The concept has been around since around 1985 with "Minimum Disclosure Proofs of Knowledge." Protocols for allowing a “prover” to convince a “verifier” that he knows some verifiable secret without allowing the verifier to learn anything about the secret.

This is useful for computer networks since eavesdroppers cannot steal the knowledge given. Useful for proving possession of some property, or credential, such as age or voting status, without revealing personal information. A practical example is user passwords, which are hashed before transmitted through the network, and thus proves possession of it without disclosing any part of it.

A zero-knowledge proof must satisfy three properties:

- Completeness: if the statement is true (e.g. I have the password), the honest verifier will be convinced of this fact by an honest prover.
- Soundness: if the statement is false (e.g. I don’t have the password), no cheating prover can convince the honest verifier that it is true, except with some small probability.
- Zero-knowledge: if the statement is true, no verifier learns anything other than the fact that the statement is true.