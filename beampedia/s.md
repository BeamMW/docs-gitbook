# S

## SBBS

In Mimblewimble there are no addresses, and transactions need to be built interactively by the participating parties. This requires that both parties set direct socket connection for every transaction, which can be impractical, and doesn’t allow offline transactions.

A Secure Bulletin Board (SBBS) system solves this problem, enabling offline transactions, which makes the user experience the same as other cryptocurrencies such as Bitcoin. With SBBS, client wallets can exchange messages by store-and-forward nodes. The messages are encrypted with public key cryptography, over elliptic curve cryptography (ECC) algorithms.

Originally, a bulletin board system is a computer server running software that allows users to connect with a client. Once logged, the user can upload or download messages through public message boards or private email, as well as file downloads (often text files), and if the client allowed, direct chatting and even text-based multiplayer games. This is the ecosystem where public key cryptography, such as Phil Zimmerman’s PGP implementation originally spread.

The architecture of a BBS is replicated with Beam’s nodes and wallets, as described in the [GitHub repository](https://github.com/BeammW/beam/wiki/Secure-bulletin-board-system-(SBBS)), and a [Medium article](https://medium.com/beam-mw/the-secure-bulletin-board-system-sbbs-implementation-in-beam-a01b91c0e919). With SBBS, messages are private when addressed to an individual, such messages are encrypted with a public key before they are sent, without having to trust relaying nodes. Other messages can be set to public and be unencrypted or sent to specific channels.

## Scriptless Scripts

Scriptless Script technology allows the blockchain to implement a variety of transaction types. This technology includes support for atomic swapping, escrow, and time-locked transactions and more.

Atomic swapping enables users to exchange cryptocurrencies without a middle-man or exchange. Furthermore, atomic swapping is faster and cheaper than centralized alternatives.

In addition to atomic swapping, this technology allows escrow transactions. Escrows are commonplace in over-the-counter (OTC) cryptocurrency transactions. Escrows allow the buyer and seller to verify funds and thus, a secure transaction. Using this technology buyers and sellers can initiate escrow transactions safely and reliably.

Time-locked transactions are exactly what they sound like: transactions that will not go through until a time condition is met. Cryptocurrency based time-locked transactions have merit as no single party can alter terms, one of the benefits of decentralization.