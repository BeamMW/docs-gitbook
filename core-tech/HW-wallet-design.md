### Beam vs other blockchains

Beam HW wallet supports the functionality similar to other blockchain protocols, such as keeping all the secret keys within the device, and signing transactions authorized by the users.

Since Beam is based in MW, the design of such a wallet is more complex, because of the following:
* Transactions are built interactively
* No addresses
* No transactions in the common sense. Transaction elements can be added and removed

## Assumptions

1. The HW wallet is a minimalistic stateless device. It should not handle blockchain events (blocks, headers, reorgs), handle BBS traffic, fully negotiate transactions, and etc.
1. The host may be compromised, yet no spend should be possible without user's permission.
1. Once the user activates the HW wallet (enters the pin-code), the host can view all the user information (balance, UTXOs), but not spend funds.
1. Every spend (transaction that assumes user spends funds) needs a user authorization. The user sees the following:
   1. The amount being-sent
   1. Asset type (beam or CA).
   1. Recipient identifier.
   1. Transaction kernel ID.
   1. Height range (min/max height) during which the transaction is valid.
   * All this information is cryptographically verified, and there should be no feasible way for the host to tamper with it.

**Note:** Because in MW transactions are essentially (sort of) Schnorr's multisignatures, the (1) is actually tricky to implement. To create a Schnorr's signature the signer generates a _nonce_, which must be retained for the duration of the multi-signature ritual.

We solve this by assuming the device has a limited non-volatile memory that can be used to store _nonce_ preimage (source material). We make sure that neither the _nonce_ nor the appropriate secret key can be extracted by malicious host (such as attempt to use the same _nonce_ to sign different things).

The HW wallet device should have several 256-bit _slots_ for _nonce_ preimages. The more slots it has - the more ongoing transactions the HW wallet may handle at the same time.

(There is also an option of using VRFs, but it's too complex for HW wallets.)

## Terminology

* `CoinID` - full UTXO identifier. Consists of the following:
   1. `SubIdx` 32-bit _child_ key identifier
   1. `Idx` 64-bit coin number
   1. `Value` (64 bits)
   1. `AssetID` (32-bit number)
   * For each `CoinID` the HW wallet generates a unique _blinding factor_ using the master secret.
* `OwnerKey` - derived from the master secret, used to recognize the owned UTXOs.
* `P-Kdf` - public key generator, which for a `CoinID` generates an appropriate public key (EC point) that corresponds to its _blinding factor_.
* `IdentityNumber` 32-bit number
* `Identity` - public key that corresponds to the unique private key generated for specific `IdentityNumber`.
* `PaymentProof` - Schnorr's signature, signed by the receiver, that it receives funds as a result of the transaction. The signature signs the following parameters:
    * Value received
    * Asset type
    * Kernel ID
    * Sender `Identity`

**Note:** the actual key generation is according to HKDF (rfc-5869). The generated _blinding factor_ depends in a non-transparent way on **all** the fields of `CoinID`, thus there's no feasible way to tamper with them and adjust the resulting _blinding factor_ accordingly.

In addition to HKDF, each computed key is multiplied by a fixed secret scalar, so-called _co-factor_. The `P-Kdf` contains the same seed HKDF data, but the _co-factor_ is replaced by its image (EC point).

**Note (2):** To denote the UTXO Beam actually uses so-called _switch_ commitments, which are compressed el-Gamal commitments. So that in practice the obtained _blidning factor_ gets an addition, which depends on the committed value and asset type. The host can calculate this addition alone (without the HW wallet).

# Functionality high-level

All the HW wallet functionality comes down to the following methods.

The following is returned without user permission
* Number of slots available
* `OwnerKey`
* `P-Kdf` for arbitrary _child_ key
* Rangeproof for arbitrary `CoinID`
* signature for receive transaction kernel (i.e. transaction in which no funds are lost).

The following needs user permission
* signature for send transaction kernel
* signature for split transaction kernel (i.e. only transaction fee is lost)

During the HW wallet initialization it generates a seed using the true random generator (available on most HW wallets), which is used to set initial values in all the slots.

# Why this design is secure?

MW transaction consists of 3 things: inputs, outputs, and transaction kernels.
* Input is denoted by its Pedersen (switch) commitment. The host can create it for any `CoinID`, since only _blinding factor_ <u>image</u> is needed, and `P-Kdf` is exported freely.
* Output consists of Pedersen (switch) commitment and the rangeproof. The commitment is calculated by the host (as for inputs), and the rangeproof is calculated by the HW wallet without any restriction.

So, both inputs and outputs can be created by the host without any restrictions. But to build a valid (balanced) MW transaction the corresponding transaction kernel is required, and this is where HW wallet restrictions come into play. As we mentioned, HW wallet requests user permission along with the verifiable recipient ID for any spend transaction.

## How do we verify the transaction is indeed spending/receiving, or splitting?

For all transactions the host specifies the input/output `CoinID`s that belong to the user before/after the transaction. Based on them the HW wallet deduces:
1. Overall value and asset which is obtained/lost in a transaction
1. excess _blinding factor_ (diff of input/output _blinding factors_).

The transaction type is verified according to (1), whereas the signed kernel balances the transaction according to (2). As we mentioned, by _blinding factor_ depends on the `CoinID` in an opaque way, hence there's no feasible way to substitute different inputs/outputs in the transaction after it was signed.

# Transaction signing

For all the mentioned transaction types the host supplies the following:
* List of input/output `CoinID`s.
* Kernel lock height (min/max height)
* Kernel fee

Upon signing, as usual, the excess _blinding factor of the transaction is split into 2 parts in a pseudo-random way: one goes to the kernel commitment, the other goes to the _offset_ (plain scalar). This is done for all the MW transactions for a better obfuscation.

Send and Receive are mutual transactions, that follow the following ritual:
1. Sender creates its part of the kernel commitment, and sends it to the receiver
1. Receiver adds its part of the commitment, signs it, and sends it back to the sender.
1. Sender finishes its part.

Split transaction (where the user sends funds to itself) is signed in a single invocation.

### Split transaction

This is the simplest case, as there's no additional transaction party. The HW wallet analyzes the input/output `CoinID`s to make sure no funds "disappear". Means the user only spends the transaction fee.

If everything matches and the user authorizes it - the HW wallet generates the kernel _blinding factor_ in a deterministic way, and then generates the kernel commitment + signature, and the appropriate _offset_.

### Receive transaction

As usual the HW wallet analyzes the input/output `CoinID`s to make sure it receives funds.

**Note:** currently it's allowed to receive only one asset type at a time, i.e. exchanging different assets in the same transaction is currently not allowed (as right now we don't have appropriate `PaymentProof` for this).

According to the ritual, by the time of invocation the sender has already produced its part. This includes the following:
* Kernel commitment
* Kernel signature "public nonce"

If everything matches - the HW wallet generates its part of the kernel _blinding factor_ in a deterministic way, and then generates its part of the kernel commitment and signature, and the appropriate _offset_.

In addition to this, the HW wallet also signs the `PaymentProof`. The Kernel ID, value and asset type are deduced automatically. The following additional parameters are specified by the host for the signature:
* Sender `Identity`
* Self `IdentityNumber`, which corresponds to the `Identity` expected by the sender. The HW wallet generates the appropriate private key, and signs the `PaymentProof` by it.

### Send transaction

This is the most complex case. To sign the send transaction the HW wallet should be invoked twice (as can be seen according to the ritual). It also needs a _nonce_ that should retain valid for the duration of the transaction.

When the HW wallet is invoked to sign the send transaction, the host, among other things, specifies the nonce _slot_ which should be used to this transaction (the slot management, i.e. which ongoing transaction uses which slot - up to the host).

On the first invocation the HW wallet generates the kernel parameters (commitment and public nonce) based on the visible parameters and this nonce.

On the second invocation it verifies receiver part, verifies the `PaymentProof` signed by the receiver, and asks for user permission to transfer the funds to this receiver. If everything is ok - the kernel is signed.

**Note:** just before signing, the HW wallet re-generates the value of the _slot_ used in this transaction (re-generates its value in a deterministic way). This is to prevent malicious host make HW wallet sign different transactions using the same _nonce_.

## Note regarding rangeproof generation

Beam uses a bulletproof as its rangeproof, which is computationally heavy for a device with limited capabilities. To improve performance the host and the HW wallet perform a multi-party computation, dividing the work by **what depends on the secret _blinding factor_ and what does not**:

* Everything that does not need the secret is derivable from the exported owner `P-Kdf`, so the host performs the bulk of the heavy elliptic-curve computation on its own — in particular the [inner-product argument](core/Core-Cryptographic-Primitives.md) (the logarithmic `L,R` recursion), which is the largest part of the proof.
* The HW wallet contributes only the secret-dependent parts: its share of the `T1,T2` commitments and the scalar that masks the _blinding factor_ with the proof challenges (`tauX`).

So far this is just a performance optimization. But a _valid_ rangeproof is not enough — the resulting output must also be **recoverable by the user's `OwnerKey`**. The blockchain only enforces that an output is valid (in range) and that the transaction balances; it does not, and cannot, enforce that the rightful owner can detect and later spend the output. A compromised host could thus craft a perfectly valid output that the user can never recover — the funds leave the wallet and are irrecoverably lost, while every full node still accepts the transaction.

To prevent this, the HW wallet does not trust the host with the recovery-bearing part of the proof; it re-derives it from the genuine seed itself. This is why — even though it offloads the inner-product argument — it still performs a multi-exponentiation of **129 generators** (the `S` commitment). Doing so binds the output to the owner's seed and guarantees recoverability. It is a considerable computation, yet only a fairly small fraction of the whole proof.

The same reasoning governs sending to a **shielded** output ([Lelantus pool](transactions/Transactions-Lelantus-Shielded-Pool.md)), where the host builds the _entire_ bulletproof and the HW wallet only signs the kernel. Before signing, the HW wallet **recovers** the host-built proof against the seed derived from the recipient's voucher — it folds the seed-derived `S`-vectors through the proof challenges, and deliberately does _not_ recompute or re-verify the `L,R` pairs (soundness is every full node's job). This recovery checks two things:

1. **Recoverability** — that the output opens from the recipient's seed, i.e. the receiver will actually be able to detect and spend it.
1. **Authentic attribution** — the bulletproof also carries data the receiver decodes: the sender `Identity` and an optional short message. The HW wallet verifies these are the authorized values, so a compromised host cannot make the payment appear to come from a different sender (impersonation, i.e. a forged `PaymentProof`).

The common principle: on-chain validation covers soundness and balance, while the HW wallet — the only component trusted against a possibly compromised host — is responsible for the properties the chain cannot see: that every output it helps create is **recoverable by its rightful owner** and carries **genuine authorship**.

## Note regarding `PaymentProof` and BBS addresses

Normally wallets negotiate over BBS system, each has a BBS address key, by which it tries to decrypt all the BBS traffic and chase the messages intended to it. Then during the transaction the BBS public address is also treated as the peer `Identity`, i.e. the sender expects the receiver to sign the `PaymentProof` by it.

However for the HW wallet it's not feasible to decode all the BBS traffic. Hence we decided to separate the BBS and the `Identity`. Now in order to send/receive funds the users exchange _Tokens_ (over secure channel), which optionally include both the BBS address and the `Identity`.

Conceptually the BBS address belongs to the entity you are directly talking to, whereas the `Identity` belongs to the <u>final</u> sender/recipient of the funds. In other words, the software wallet is a proxy, which may be compromised, whereas still there is a confirmation that the final recipient gets all the funds.
