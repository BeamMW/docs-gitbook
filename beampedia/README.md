# Beampedia

Beampedia is an encyclopedia of key terms and concepts related to Beam, a privacy-focused cryptocurrency built on Mimblewimble. It provides accessible explanations of Beam's technical underpinnings and ecosystem. Whether you're a beginner seeking to learn the basics or an expert looking to dive deeper, Beampedia aims to be a go-to reference for all things Beam. The entries cover a wide range of topics - from Beam's origins and goals, to the cryptography it leverages, the network architecture and components, terminology around usage and investment, details on software and configuration, and more. Beampedia serves as a knowledge base to empower users to fully utilize Beam's confidential transactions, understand its advances in areas like transaction privacy and scalability, and participate in the broader vision of an open, decentralized cryptocurrency ecosystem.

## A

### Account

Similar to what is commonly referred to as a wallet an _account_ in Beam is a private account owned and operated by the user.

An account contains all of the user's transaction history, as well as the resulting current total _balance_.

A Beam account is [self-sovereign](https://www.ibm.com/blogs/blockchain/2018/06/self-sovereign-identity-why-blockchain/), which means that only the users can access and control their _balance_ and that the full responsibility rests on them. The account is usually stored locally on the user's device or remotely using encrypted cloud storage for backup.

This is achieved with _public key cryptography_, which uses key pairs: a public and a private key. The public key can be shared, and it can be used by the user to view the _balance_ without any security risks. The private key must be stored securely, and is only used for signing transactions. 

### Address 

A Beam _address_ is a unique identifier derived from a user's public key that can be shared to receive beam payments. Addresses in Beam utilize _Confidential Transactions (CT)_, which encrypt transaction amounts in the address itself. This provides privacy by concealing the account balance and transaction amounts. 

New addresses should be generated for each transaction to enhance anonymity. As addresses are free to create, this impedes blockchain analysis intended to cluster addresses.

### Architecture

Computer architecture involves designing computing systems to optimize for functional, performance, cost, reliability and efficiency goals. This requires making hardware and software design tradeoffs. 

Architectural improvement is driven by advances like Moore's Law, allowing transistors to be miniaturized at an exponential rate. This enables new applications and capabilities. However, unlimited scaling is impossible, so disruptive changes are eventually needed. 

For example, the emergence of multicore CPUs, SSD storage, and new algorithms like public key cryptography have enabled modern systems. Ongoing research into areas like quantum computing aims to drive future disruptive improvement.

### Atomic Swaps

Atomic swaps allow direct cryptocurrency trades between users without a centralized exchange. Trades occur via smart contracts that either atomically complete or abort, preventing theft.

Benefits include decentralization, privacy, security, and low fees. Users retain custody of funds and transact peer-to-peer. This is especially useful for swapping privacy-focused cryptos like Beam in a trustless manner.

As technology improves, atomic swaps will become faster and more decentralized. This can facilitate an open, global crypto economy without intermediaries.

## B

### Beam 

Beam is a privacy-focused cryptocurrency that utilizes Mimblewimble to provide confidential transactions by default. This allows for scalability, reduced blockchain bloat, and enhanced fungibility without trusted setup.

Beam operates on a proof-of-work model using the ASIC-resistant Equihash algorithm. Periodic halving and an initial treasury emission provide long-term sustainability. The developers have experience building performant C++ systems.

Beam implements leading research across areas like Confidential Transactions, Bulletproofs, and Dandelion++ for privacy and scalability. It enables complex logic like time-locked transactions and atomic swaps. Beam aims to be a decentralized, confidential cryptocurrency for everyday transactions.

### Block

In blockchain networks, a block contains a batch of transactions to be verified and added to the chain. Blocks have a maximum size and contain a cryptographic hash of the previous block to link together.

Valid blocks must follow the network's consensus rules. Nodes express acceptance by building on top of blocks, while invalid blocks are rejected. The mining process creates new blocks by solving a computational puzzle. 

In Bitcoin and Beam, blocks contain a Merkle root hash of all transactions. This both batches transactions and establishes their order to prevent double spends. Blocks form the fundamental structure of blockchain ledgers.

### Blockchain 

A blockchain is a decentralized ledger achieved by cryptographic linking of transaction batches (blocks) in an append-only data structure. This prevents revision and establishes an authoritative transaction history.

Satoshi Nakamoto proposed using proof-of-work and economic incentives to operate blockchains in a trustless way. Network nodes cryptographically agree on the valid chain with the greatest proof-of-work. Technologies like Mimblewimble improve privacy and scalability.  

While originally conceived for Bitcoin, blockchains now have many applications requiring tamper-resistant state and consensus without intermediaries. Active research is still improving their capabilities.

### Bulletproofs 

Bulletproofs are a type of zero-knowledge proof that enable confidential blockchain transactions without trusted setup. They are succinct proofs that transaction amounts are valid under certain rules.

For privacy coins like Beam, they replace expensive range proofs that were previously required to verify confidential transaction amounts. This significantly reduces proof sizes and blockchain bloat.

Bulletproofs leverage discrete log equivalence and other innovations to provide efficient confidential transaction validation. They are an active research area for both blockchain privacy and zero-knowledge applications.

## C

### C++

C++ is a general-purpose, compiled, statically typed, object-oriented programming language developed by Bjarne Stroustrup at Bell Labs starting in 1979, originally as an extension of the C programming language. 

C++ was designed for flexibility, efficiency, performance and speed, making it well-suited for system programming, embedded systems, resource-constrained devices, and large-scale software development.

Key features and paradigms in C++ include:

- General-purpose - can be used to build a wide array of applications, unlike domain-specific languages.
- Imperative - uses statements that change program state, unlike declarative languages which specify desired outcomes.
- Object-oriented - structures programs around objects and classes instead of just functions and logic.
- Low-level memory access - allows direct manipulation of memory addresses and pointers for performance.
- Compiled - source code must be compiled to machine code, unlike interpreted languages. 
- Statically typed - all types must be known at compile time for efficiency and safety.

Later versions of C++, such as C++17 used in Beam, add features like type inference, lambda expressions, templates, and reflection.

### Confidential Assets

Confidential Assets is a Mimblewimble extension allowing the blinding of asset tags in transactions to preserve privacy. It enables multiple asset types to be transacted confidentially on a blockchain without exposing users' balances or behaviors.

There are two types - predefined and user-defined. Each has tradeoffs between privacy and expression.

### Confidential Transactions 

Confidential Transactions, proposed by Adam Back, hide transaction amounts using cryptographic commitments that preserve the ability to verify no coins were created or destroyed. 

Only transaction participants can see the actual amounts, while the network can still validate the transaction is balanced. This provides transaction privacy without introducing new cryptographic assumptions.

Implemented by Blockstream, Confidential Transactions also enable private memo data to be exchanged with no additional overhead.

### Consensus 

In distributed systems like blockchains, consensus refers to nodes agreeing on a single ordered set of valid transactions, despite failures. 

Consensus algorithms must satisfy termination, integrity, and agreement properties to be considered fault tolerant:

- Termination - all non-faulty nodes decide on a value 
- Integrity - can recover from node failures
- Agreement - all non-faulty nodes agree on the same value

In Bitcoin, consensus is achieved via proof-of-work and the longest valid chain rule (Nakamoto consensus). Miners build on top of the chain they consider canonical.

### Cryptocurrency

Cryptocurrencies are digital assets and protocols that allow electronic value transfer. Early systems focused on anonymity, attracting interest from illegal markets, but also enabling privacy.

Debates continue around their relationship with national currencies and regulations. However, anonymous digital cash is important for individual privacy and empowerment. 

While previously attempted, Bitcoin sparked mainstream crypto interest by solving key technical challenges. Beam builds on this progress with advances like Mimblewimble and Equihash.

### Cryptographic Signatures

Cryptographic signatures prove authenticity of messages using public key cryptography. The signature depends on the message content and a private key only held by the signer. 

Anyone can verify the signature's validity without revealing the private key. Digital signatures are a basic requirement for trust over the internet.

## D

### Dandelion

Dandelion is a transaction routing protocol that enhances privacy against network surveillance and deanonymization attacks. 

It was proposed by Giulia Fanti et al. and is based on Bitcoin's BIP-156. Dandelion mitigates transaction source identification by transmitting transactions silently along a random path during a stem phase, before diffusing it to the wider network during the fluff phase.

Without Dandelion, transactions diffuse symmetrically from the source, allowing spy nodes to triangulate the source IP based on transaction propagation. Dandelion protects the anonymity of the entire network, rather than just individual users.

### Data Directory

The data directory is the disk location where Beam node and wallet data is stored, including:

- Mac: `/Users/{username}/Library/Application Support/Beam Wallet/`  
- Windows: `\Users\{username}\AppData\Local\Beam Wallet`
- Linux: `/home/{username}/.local/share/Beam Wallet`

Upon starting a node, log files are created in the logs subfolder. The node will create a node.db file storing its internal state. 

On first launch, the node syncs the full blockchain history in batch mode as a macroblock. After initial sync, it will continue syncing new blocks and transactions from the blockchain tip onwards.

## E

### Elliptic Curve Cryptography

Elliptic curve cryptography (ECC) is a modern approach to public-key cryptography based on the algebraic properties of elliptic curves. 

An elliptic curve is the set of points satisfying the equation y^2 = x^3 + ax + b. Adding points on the curve yields a third point, allowing cryptographic operations. 

Encrypting a message maps it to a point, which is then repeatedly added to itself k times. Without knowing k (the private key), decryption is infeasible. 

ECC provides exponential security improvement over RSA with smaller keys. It is standardized by NIST and widely adopted in protocols like TLS.

Beam uses the secp256k1 curve, providing 128 bits of security. The large prime field size hinders brute force attacks.

### Emission

Emission refers to the creation and distribution of new coins in a cryptocurrency system as a mining reward.

In Beam, emission is smooth and predictable based on a fixed block reward that halves periodically. This incentivizes mining while controlling inflation.

As the block reward decreases over time, transaction fees will become a larger portion of miner income, providing an incentive to keep securing the network.

### Encryption

Encryption encodes messages to prevent unauthorized access. Symmetric encryption uses a shared secret key. Asymmetric public key encryption uses public keys for encryption and private keys for decryption.

Public key cryptography enabled e-commerce by providing authentication without prior shared secrets. RSA is the most common algorithm. Elliptic curve cryptography (ECC) provides stronger security per bit.

Beam uses ECC due to its efficiency and high security margin against brute force attacks.

### Equihash 

Equihash is a memory-hard proof-of-work mining algorithm developed by Alex Biryukov and Dmitry Khovratovich.

It is based on the generalized birthday problem of finding collisions between sets of random values. The algorithm parameters n and k determine the memory and bandwidth required.

Zcash implemented Equihash, but Bitmain produced an efficient ASIC miner for it. While ASIC resistance is ultimately futile, Beam will use Equihash parameters that give CPUs and GPUs a head start.

This allows widespread coin distribution while still transitioning to the security of ASICs later. Finding the right balance enables a democratic distribution without sacrificing decentralization.

## F

### Financing

Financing provides funds for business activities through debt, equity, or token offerings. 

- Debt - Loans that must be repaid with interest.

- Equity - Ownership stakes given to investors.

- Tokens - Blockchain-based assets sold to raise funds. Can represent utility, securities, or currencies.

Utility tokens leverage network value but are risky due to speculation. Regulated security tokens are emerging as a more stable model. 

Beam's Confidential Assets allow private token issuance, preserving issuer and investor privacy.

### FOMO

FOMO stands for "fear of missing out," describing anxiety over missing rewarding experiences or opportunities. 

FOMO stems from anticipation of regret and social comparisons. Behavioral economics studies factors like FOMO that contradict rational decision theory. 

FOMO is a form of reference dependence - evaluating outcomes based on comparisons rather than absolute values. It highlights the role of psychology and emotion in economic behaviors.

### Fungibility

Fungibility is the property of assets being interchangeable with no loss of value. Fungible assets can be freely substituted for one another.

Currency is fungible - one dollar bill equals any other. Gold is fungible if of the same purity and weight. Diamonds are non-fungible since each one is unique. 

Fungibility does not require liquidity. Diamonds are liquid but not fungible.

In cryptocurrencies, fungibility depends on transaction privacy. With a public ledger like Bitcoin, some coins can become "tainted" by their transaction history. This makes them less accepted, damaging fungibility.

Tainted coins can unknowingly spread, harming innocent holders' reputations. Privacy protections like Confidential Transactions are essential for cryptocurrency fungibility. When all coins are indistinguishable, they remain interchangeable without stigma.

## G

### Groth 

A Groth is the smallest denomination of Beam cryptocurrency recorded on the blockchain. 

1 Groth = 0.00000001 BEAM (one hundred millionth of a Beam).

The unit is named after Jens Groth, Professor of Cryptology at University College London. 

Prof. Groth's work on efficient cryptographic proofs and arguments is fundamental to Mimblewimble and other privacy-focused cryptocurrencies. 

Naming the smallest unit after him honors his contributions to the development of private and scalable blockchain technology.

## H

### Hash

A cryptographic hash function maps data into a fixed-size string or fingerprint. Ideal properties:

- Deterministic - same input gives same hash
- Quick to compute
- Infeasible to invert 
- Small changes to input alter output completely
- Resistant to collisions

Common algorithms include MD5 (deprecated), SHA-1 (phased out), SHA-2, SHA-3, and BLAKE. 

BLAKE is optimized for 64-bit CPUs and used in Beam's Equihash mining. Hash functions enable data integrity checks.

### Hold

"Hold" refers to a long-term passive investment strategy of buying and retaining a cryptocurrency regardless of price fluctuations.

It originated from a 2013 bitcointalk post during high volatility. The rationale was inexperienced traders often lose money trading while seasoned investors hold through bear markets. 

"Hodl" became a meme for believing in the long-term potential of a cryptocurrency as sound money rather than short-term gains.

However, hold advice around "pump and dump" schemes can be malicious, trapping naive investors. The aim of cryptocurrency is a store of value, not speculative investment.

True believers hope for fiat currency hyperinflation and the ultimate adoption of cryptocurrency for all transactions.

## L

### Lambo

"Lambo" is slang for Lamborghini, an Italian luxury sports car brand known for speed and expense. 

The meme arose from cryptocurrency enthusiasts imagining buying Lamborghinis with their crypto gains. It represents hopes of rapidly accumulating life-changing wealth.

In 2010, the first Bitcoin pizza purchase valued BTC at a fraction of 1 cent. By 2017, 1 BTC was worth 1/20th of a Lamborghini Aventador. 

"When Lambo?" became shorthand for "When will my crypto be worth enough for extravagant purchases?" It is the speculative equivalent of "To the moon!"

However, realizing Lamborghini dreams requires responsible tax planning. Cryptocurrency windfalls invite regulatory scrutiny, particularly for those not reporting commensurate income.

While evocative, the Lambo meme underestimates literal challenges of moon buggy transport and the lack of a breathable atmosphere.

### Lelantus

Beam utilizes a hybrid of Mimblewimble and Lelantus to enhance transaction graph privacy. 

Lelantus is a protocol developed by the Zcoin team. It allows creating a shielded pool of anonymous UTXOs without trusted setup.

By combining Lelantus with Mimblewimble, Beam achieves a large anonymity set and breaks links between UTXOs. This makes transaction tracing virtually impossible.

Lelantus also enables one-sided payments in Beam, removing Mimblewimble's interactivity requirement for sending/receiving.

Overall, Lelantus-MW is a major advancement for transaction privacy in Beam, preventing the graph analysis possible in basic Mimblewimble.

Read the full explanation on Lelantus [here](https://github.com/BeamMW/beam/wiki/Lelantus-MW).

## M

### Merkle Tree

A Merkle tree is a cryptographic data structure invented by Ralph Merkle in 1979. It allows efficient verification of large data sets.

Merkle trees produce a root hash from recursive hashing of paired data. This allows verifying membership in the tree with just a branch, not the full tree. 

Merkle trees provide an efficient way to encode blockchain transaction data. Clients can verify transactions without downloading the entire chain.

### Mimblewimble 

Mimblewimble is a blockchain protocol designed for privacy and scalability. It was [proposed anonymously by pseudonymous Tom Elvis Jedusor in 2016](/mimblewimble.txt) as a radical change to Bitcoin.


The name comes from a Harry Potter tongue-tying curse, reflecting its goal of preventing the blockchain from leaking user information.

Mimblewimble transactions are aggregated without interactivity via Confidential Transactions and CoinJoin. 

Blocks contain input, output, and excess lists rather than complete transaction data. This provides privacy while allowing nodes to verify no coins were created.

By eliminating scripts, Mimblewimble increases scalability but loses functionality compared to Bitcoin. Beam implements extensions like scriptless scripts to regain expressiveness.

### Mining 

Mining is the process of adding transaction records to a cryptocurrency's blockchain through proof-of-work computation. It serves to:

- Generate new coins via the block reward
- Distribute coins into circulation  
- Confirm transaction history
- Secure the network through economic incentives

Miners use hardware optimized for the proof-of-work algorithm:

- CPUs - Earliest mining, now obsolete 
- GPUs - More parallelism than CPUs
- FPGAs - Custom hardware, outpaced by ASICs
- ASICs - Application-specific chips built for mining

To smooth reward variance, miners often join pools to share proceeds. 

Algorithms like Beam's Equihash are memory-hard, resisting ASIC optimization over GPUs for a more decentralized distribution.

### Moon

The Moon is Earth's only natural satellite and the first celestial body visited by humans. Its phases result from its 27-day orbit around Earth. The Moon causes ocean tides due to its gravity. Tidal forces also slowed the Moon's rotation, locking one face toward Earth.

The leading theory is that the Moon formed from debris after a collision between Earth and a Mars-sized body called Theia 4.5 billion years ago.

In cryptocurrency contexts, "to the moon!" refers to hopes for massive price increases, often fueled by coordinated pumping. Inexperienced traders use this phrase when trying to profit from speculative manias around low market cap "shitcoins."

While evocative, the metaphor overlooks challenges like the Moon's lack of atmosphere and 238,000 mile distance from Earth. Responsible investing should be based on more than aspirational slogans.

## N

### Node

In graph theory, a node is a fundamental unit of a graph, connected to other nodes by edges. 

In computing, a node refers to devices on a network or data points in a data structure. Nodes have identifiers like IP addresses.

In distributed systems like blockchains, nodes are peer computers that maintain shared state (the ledger) without a central clock. 

A cryptocurrency node validates transactions and blocks, enforces consensus rules, and relays data. Nodes can join or leave the network at will.

Mining nodes create new blocks, while full nodes validate the entire blockchain. Lightweight nodes only validate their own transactions.

Consensus emerges through nodes independently verifying and extending the canonical chain. Nodes communicate on a best effort basis without centralized routes.

## O

### Offline Transactions

Mimblewimble requires sender and receiver interactivity to construct transactions. Beam's Secure Bulletin Board System (SBBS) removes this limitation, enabling offline transactions.

With SBBS, nodes relay encrypted messages, facilitating transactions without direct connections. This provides a similar user experience to Bitcoin.

Messages are private when addressed to an individual public key. Public unencrypted channels are also possible. SBBS allows Mimblewimble scalability without sacrificing convenience.

### Opt-in Auditability 

Auditability is a Beam feature allowing businesses to prove transaction history to auditors while maintaining privacy. 

Users can generate an auditing key pair. The wallet then includes additional info to classify transactions.

Auditors cannot initiate transactions or spend funds. But with the user's consent, they can retrieve full transaction details from the wallet and blockchain.

Auditability is strictly opt-in. If not explicitly enabled by a user, transaction details remain completely private. This balances compliance needs with user privacy.

## P
### Pedersen Commitment

A Pedersen commitment allows committing to a value without revealing it, while still enabling validation later. 

Commitments are constructed by combining a blinding factor with the data and hashing them. This hides the data while binding to it. 

Pedersen commitments have a useful additive property - the sum of commitments equals the commitment to the sum. This enables confidential transactions.

Commitments prevent external parties from inferring the hidden data. The blinding factor provides security even for small values. Pedersen commitments are fundamental to Confidential Transactions.

### Privacy 

Privacy is preventing unauthorized extraction of information in communications. Encryption protects privacy just as locked doors prevent unwanted entry.

While not enumerated explicitly, privacy is assumed as a right in liberal democracies. Wiretaps and invasive surveillance require judicial approval.

Privacy is tied to fungibility in cryptocurrency. Public ledgers like Bitcoin risk "tainting" certain coin histories, damaging fungibility. 

Unlike opt-in systems like Zcash, Beam provides default privacy for all transactions. This is essential for maintaining strong currency fungibility and ethical sound money.

## S

### Secure Bulletin Board System (SBBS)

Mimblewimble requires sender-receiver interactivity to construct transactions. Beam's SBBS removes this limitation.

SBBS allows clients to exchange encrypted messages via relay nodes. This facilitates transactions without direct connections.

Messages are private when encrypted to a public key. Public unencrypted channels are also possible.

SBBS replicates old bulletin board systems where public key cryptography spread, enabling decentralized discussions.

By relaying encrypted data, SBBS provides Mimblewimble's scalability without sacrificing convenience or privacy. It matches the user experience of transparent blockchains.

See Beam's [GitHub](https://github.com/BeammW/beam/wiki/Secure-bulletin-board-system-(SBBS)) and [Medium post](https://medium.com/beam-mw/the-secure-bulletin-board-system-sbbs-implementation-in-beam-a01b91c0e919) for technical details.

### Scriptless Scripts

Scriptless scripts enable advanced transaction types in Beam without a scripting language.

Techniques like Schnorr signatures and key aggregation allow features like: 

- Atomic swaps - Trustless cryptocurrency exchange between parties.
- Escrows - Holding funds until conditions are met, useful for OTC trades.
- Timelocks - Transactions that only confirm after a certain time.
- Multisig - Transactions requiring multiple signers.
- Oracles - External data triggers contract conditions.

By innovating cryptography like threshold signatures, Beam gains functionality without the overhead of a scripting language like Bitcoin's.

Scriptless scripts provide expressiveness equivalent to Ethereum but with Mimblewimble privacy and scalability. This makes advanced transactions accessible to regular users.

## W

### Wallet

A cryptocurrency wallet manages the keys used to sign transactions on the blockchain. 

Wallets store public/private key pairs. The private key signs transactions to prove the spender's identity. 

The wallet itself does not hold coins - it is software that displays balances from blockchain data.

Wallets can be:
- Nondeterministic - Randomly generated keys
- Deterministic (HD) - Keys derived from a master seed

HD wallets use mnemonic phrases for usability and security. 

Wallets allow users to securely interact with the blockchain, acting as a gateway to send, receive, and monitor funds.

Beam wallets store data in default user data folders based on the operating system:

- Mac: `/Users/{username}/Library/Application Support/Beam Wallet/`  
- Windows: `\Users\{username}\AppData\Local\Beam Wallet`
- Linux: `/home/{username}/.local/share/Beam Wallet`

## Z

### Zero-Knowledge Proof

A zero-knowledge proof allows one party (the prover) to convince another (the verifier) that a statement is true without revealing any information beyond the validity of the statement. 

Examples include proving knowledge of a password without disclosing it, or proving eligibility to vote without identifying personally.

Zero-knowledge proofs have three properties:
- Completeness - An honest prover convinces an honest verifier if the statement is true.
- Soundness - No dishonest prover can convince an honest verifier of a false statement. 
- Zero-knowledge - The verifier learns nothing but the statement's validity.

This ability to validate facts privately is essential for blockchain privacy. Mimblewimble transactions utilize zero-knowledge proofs to ensure no new coins are created while hiding transaction details.
