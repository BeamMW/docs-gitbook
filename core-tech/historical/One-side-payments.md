> **Historical Document** — This document describes an early one-side payment proposal for Beam/Mimblewimble. It is preserved for historical reference only.

In MW in order to create a valid transaction all the parties must collaborate. Here we'll present a one-side payment scheme, which, after initial setup, allows arbitrary senders to pay specified (fixed) values to a particular receiver, without any further collaboration from the receiver side.

# Principle

In order to receive a payment the receiver creates and publishes the following:
* The value that it expects to receive
* An UTXO (commitment + Bulletproof) that encodes the above value, with arbitrary blinding factor
* A *compensatory* kernel that compensates for the above blinding factor
* Optionally: a signed payment proof
   * Should include the received value, and the above kernel ID
   * No sender address, since it's unknown, plus payment may come from multiple senders
   * Should be signed by a private key bound to the receiver identity

The above UTXO and the kernel form a part of the transaction with zero (or any other <u>known</u> value) blinding factor, and a known value on output. Let's call it an *acceptor* part.

In order to pay this receiver the sender create its own transaction part, which has appropriate value on input, and the kernel that compensates for its blinding factor. Call it a *donor* part.

Together those 2 parts form a valid transaction with 2 kernels.

# Why this is not good enough

The above scheme is naïve, and there are several problems with it.

## Value can be stolen

As we said, the above transaction consists of two parts: the *donor* and the *acceptor*. Means it's <u>not atomic</u>. A malicious node may delete the original *acceptor* part, and instead substitute its own, effectively stealing the value.

If the above payment scheme was agreed secretly between the sender and the receiver - still there's a chance the receiver is colluded with the miners.

## Impossible to prove the payment

Normally the existence of the kernel in the blockchain together with the payment proof signed by the receiver unambiguously prove the payment. However in this case the signed payment proof doesn't include the sender address, and the sender can't prove that the payment was made by him/her.

# Improved scheme

In BEAM the above issues are solved by the following extensions to the protocol.

## Kernel fusion

In BEAM it's possible to create a transaction kernel which includes "internally" arbitrary number of additional ones. The kernel signature accounts for all the contents, so it's not feasible to tamper with it.

So, in order to send a payment, the sender creates a *composite* kernel, which includes *internally* the receiver compensatory kernel. This makes it impossible for the attacker to remove the receiver kernel, hence the receiver UTXO also must remain. This guarantees the integrity of the transaction.

In addition the sender now has an ability to prove that the payment was made by him/her. For this the sender must do the following:
* Locate its composite kernel in the blockchain
* Demonstrate that it includes internally the receiver compensatory kernel, as well as the payment proof signed by the receiver, which means that this receiver kernel means accepting a specific value.
* Prove that the composite kernel indeed belongs to the sender
   * It's sufficient to prove the knowledge of the blinding factor of the "outer" part of the kernel. Given an arbitrary data, the sender will sign it with its blinding factor. The verifier may check it corresponds to the visible kernel excess.
   * This scheme must be elaborated though to prevent MITM attack.

## Duplicated UTXOs are allowed in BEAM

This is important to allow multiple payments, as well as payments in terms of several instances of predefined values.

# Additional notes and caveats

## Potential problem with transaction replaying

Assume receiver owns several identical UTXOs, and decides to spend only one of them in a transaction.
The problem is that such a transaction can be replayed (repeated) several times, without the permission of the owner, each time it will consume the next such an UTXO.

To prevent this the wallet must NEVER create a transaction that spends only (potentially) duplicated UTXOs. The set of inputs must contain at least 1 unique UTXO, which is not supposed to get duplicated (with non-negligible probability).

This logic must be implemented in the wallet. In the simplest case the wallet may have a single "dummy" UTXO (i.e. with zero value) for this. Whenever necessary it will be used as an input, whereas another unique dummy will be created on output instead of the consumed one.

## Less anonimity

Naturally the described scheme provides less anonymity than the standard MW. This however impacts the receiver only, and basically only those UTXOs used in this scheme.
