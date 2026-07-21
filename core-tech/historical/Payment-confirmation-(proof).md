> **Historical Document** — This document describes the payment confirmation (proof) mechanism for early Beam wallet versions. It is preserved for historical reference only.

Before describing the Payment confirmation - some basic definitions, concepts and disambiguations.

## Coins, Transactions, Kernels and BBS addresses

In MW there are no addresses, and all the UTXOs (i.e. coins) look like random EC points, no public info that can bind a specific UTXO to a user.

Upon initialization the wallet either generates a secret 12-word phrase, or is given it explicitly (in case of restore). It is used to derive the <u>**_Master Secret_**</u> - the root secret key, used to generate all other secret objects.

### UTXOs

In Beam UTXOs are created by the wallet using its _Master Secret_, and they are signed in a special way, by which the wallet can identify them in the blockchain, whereas for others they are indistinguishable.

<u>Note</u>: when your wallet receives a payment from another one - your resulting UTXO is created by <u>your</u> wallet, not by the sender.

This means that by scanning the blockchain you can **always** recover all your UTXOs that are unspent yet. It doesn't matter how the transaction was negotiated, which BBS address was used. UTXOs can always be identified with the _Master Secret_.

### Transactions

In MW parties must negotiate to build a valid transaction. Upon successful negotiation they build and then broadcast a valid transaction, that consumes some of their UTXOs (inputs), produces new UTXOs (outputs), and a _Transaction Kernel_ (more about it later).

In Beam transactions are negotiated via (S)BBS - secure message exchange system.

### BBS address

In order to negotiate parties generate at least one _BBS address_. It consists of the following:
* Channel number
* Public Key

In order to send a BBS message to an address one uses the Public Key to encrypt the message, and then this message is broadcasted on the specified channel and replicated in all the Nodes. There is an E2E encryption, no info about to/from is leaked as well, and it's only possible to decrypt with the appropriate <u>Private Key</u> of the BBS address.

### Kernels

Each transaction produces a transaction kernel (one or more), which has the following properties:
1. It's co-signed by all the transaction parties (the sender and the receiver in a simple payment transaction). Both must cooperate.
1. It's practically unique. There is no feasible way to craft the same valid kernel without the cooperation of all the parties.
1. Unlike UTXOs it can't be consumed, hence it's guaranteed to stay in the blockchain.

So, the <u>kernel in the blockchain is the only reliable indicator of the transaction</u>. The even presence of it in the blockchain unambiguously tells the transaction took place, whereas its absence means the opposite. So all the transaction parties can monitor the blockchain (or ask for a proof from an arbitrary node) to see if/when the transaction takes place.

However in case of a dispute the <u>receiver can deny any relation to this transaction</u>.

# Payment confirmation

The _Payment Confirmation_ is a cryptographic proof, using which the sender should be able to convince a 3<sup>rd</sup> party that the receiver actually received the payment.

It's a message, signed by the receiver, that he/she accepts the payment. It includes the following:
* Value to-be-received
* `KernelID` - the unique identifier of the kernel of the transaction
* `SenderID` - The Public Key of the sender BBS address
* `Signature` - The Schnorr's signature of this message, signed by the Private Key of the receiver BBS address.

During the transaction negotiation, before the sender completes its part of the kernel signature, the receiver is expected to sign the Payment confirmation. Without it the sender will not collaborate, and the transaction negotiation fails with "no payment confirmation" error (unless unconfirmed payments are enabled, please see remarks for more info).

Upon successful negotiation - the Payment confirmation is stored within the sender wallet, and can be exported and then validated by everyone.

## Dispute - what should be presented/assumed?

In case the transaction actually took place whereas the receiver denies it - the following should be presented:
* Proof of the kernel with the specified KernelID in the blockchain.
   * Can be discovered via blockchain explorer.
   * It's also possible to get a cryptographic proof of inclusion this kernel in a specific parent block of the current tip.
   * Since blocks also include the Timestamp - it's also an indication of when the transaction took place.
* Payment confirmation.

The following is assumed:
* The receiver BBS address (Private key of which was used to sign the Payment confirmation) indeed belongs to the receiver
   * This is usually the case for automated payment systems, where a user registers via a website, and defines its BBS address for payments.
   * For regular P2P payments the sender should consider how it'd bind the specific user to its BBS address. Perhaps some documentation or message history should be kept.
* The sender BBS address indeed belongs to the sender
   * The receiver may claim that this specific transaction was from someone else
   * The sender theoretically **can** prove that the sender address belongs to him/her by proving that he/she has the appropriate private key.
      * A naive non-interactive scheme (like the one used by the receiver) is inappropriate, because it's prone to the MITM attack.
      * Right now the appropriate interactive scheme isn't implemented yet. But it will be in the near future.


## Remarks

* Initially released wallet didn't support the payment confirmations.
   * To be backward-compatible, the newer wallet by default **will** send a payment to the older one, without confirmation.
   * This can be prevented, by explicitly configuring it to disallow unconfirmed payments.
      * `beam-wallet --payment_proof_required=1`
* During wallet restore - only the unspent UTXOs are recovered.
   * Neither transaction info, nor Payment confirmations can be recovered if they are lost
   * It's important to back-up the wallet
* Exporting the Payment confirmation:
   * `beam-wallet payment_proof_export --tx_id=<tx_id>`
   * `<tx_id>` is the internal transaction ID
   * Upon success the payment confirmation summary will be printed, as well as the appropriate text-encoded (base-64) message, including all the relevant data and the signature, that can be copied and verified within an arbitrary wallet.
* Verifying the Payment confirmation:
   * `beam-wallet payment_proof_verify --payment_proof=<message>`
   * `<message>` is the exported text-encoded message.
   * The message will be parsed, signature verified, and upon success - the Payment confirmation details will be printed.
