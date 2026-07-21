> **Historical Research** — This document describes a proposal for kernel reuse via a multiplier scheme. It was never implemented in this form. It is preserved as a record of research into MW design trade-offs.

### Transaction and kernels
One of the fundamental features of a MimbleWimble transaction is the <u>irreversibility</u>. That is, once the transaction enters the blockchain, it can't be reversed, which guarantees that once a user receives the coins, the sender can't "change his mind" later.

This is achieved by transaction <u>_kernels_</u>, which are cryptographically signed, and encode (among other things) the total blinding factor of the transaction. While a UTXO can be either consumed or created by the transaction, the kernels can only be created, and never consumed. This guarantees the irreversibility of the transaction.

This leads to the accumulation of the transaction kernels in the system with time. While the UTXOs denote the "live" coins, which can be spent by their owners, the transaction kernels are actually "dead weight", and play no role in future transactions. OTOH the fully-functional processing node is obliged to keep them forever, in order to allow for new nodes to sync and verify the most recent blockchain state.

### How to consume the transaction kernels - principles

Let's assume that transaction kernels can also be consumed, and a valid transaction can contain transaction kernels on both input and output sides. In order to keep the irreversibility principle we apply additional restrictions on the output kernels, such that the verifier can:
1. Verify that for each input kernel there's a corresponding output kernel, that can only be signed if its creator(s) know the opening of the input kernel.
1. Such a transaction is irreversible "on its own". Means - input and output kernels can't be exchanged.
1. The above rules must also hold when transactions are merged and intermediate outputs are fully deleted.
1. In addition the output kernel should remain compact, and not inflate with every such a transaction. There's no benefit of keeping one large kernel vs many small ones.

# Proposal

Kernel consists of the following fields:
* Public excess: the `k*G` curve point, whereas the `k` is the total blinding of the transaction.
* Various optional fields (fee, block height, custom co-signed contract, nested kernels).
* Schnorr's signature of the kernel body, whereas the `k` is the signature private key.

The blinding factor `k` is the sum of the total blinding factors of all the participants, and the kernel is co-signed collectively.

Let's add additional field to the kernel:
* `m` - Multiplier, a 64-bit integer.

If not specified - by default it's assumed to be 1. The actual transaction excess is `mk*G`, i.e. multiplied by `m`.

During the transaction verification, the verifier must ensure that for every input kernel there is a corresponding output kernel, which:
* Has the same unmultiplied excess `kG`.
* Has a multiplier which is **greater** than that of the input kernel.

It's easy to see that such a scheme conforms to the rules 1-4.

1. In order to co-sign the new kernel, the participants **must** know the `mk`, hence they know `k`. Means - those are the same participants that signed the input kernel.
1. Such a transaction is irreversible, since the output multiplier must be (strictly) greater than the input.
1. When such a procedure repeated many times, and intermediate outputs are deleted - we have the same `kG` correspondence, and the same inequality for multipliers.
1. The kernel remains compact, its size is independent of the reuse count (up to order of 2<sup>64</sup>).

# Remarks

### How to encourage this scheme

Naturally users are not obliged to consume their old kernels, and are free to create new ones (even several kernels) with every transaction.
However we'd like to encourage this scheme. Hence we'll define an <u>**implicit kernel fee**</u>. So that consuming old kernels will be beneficial to the users.
Optionally this fee may depend on the kernel size (the size of a single kernel is limited, but it may contain arbitrary number of nested kernels).

### How to use it in practice

This scheme can be used by the same set of users (2 in the most common scenario), which perform several transactions.
On the first transaction each peaks a random `k` used to construct the first transaction.
On the consequent transactions they decide to reuse the older kernel and increment the multiplier `m` for the new kernel.

### Degrees of freedom for newly-created UTXOs

The total blinding factor of each consequent transaction is pre-defined (`kG` multiplied by the multiplier difference). It may seem that due to this fact the users are restricted to pick specific blinding factors for their UTXOs (hence - making them less confident and easier to track), but this is not so.
The transaction, in addition to the UTXOs and kernels, contains also an arbitrary `offset` - the arbitrary non-encoded blinding factor, which is summed when transactions are combined. Using this degree of freedom users can pick arbitrary blinding factors for the newly-created UTXOs.

### Confidentiality consideration

One obvious thing is that anyone can see that the same set of users do several transactions. Without dandelion a hostile node can reveal the user identity (IP address) for the specific transaction that it received. If the kernel consuming scheme is used, the hostile node can also detect when the same set of users create more transactions (both in the past and in the future), even if they used another node.
However it will not be able to identify and track the appropriate UTXOs (unless this hostile node actually received all those transactions).

With the use of dandelion this risk is reduced (but not removed completely).

I can't see other issues, and this specific one is minor IMHO.

### What if the user wants the kernel to remain forever?

As we said, the kernels are "dead weight" that the processing node normally can't get rid of. But in fact kernels may encode some information useful for users. For instance, it may be a co-signed contract, which users may use later to prove the payment.

In this specific case users should wait until the transaction becomes visible in the block, and obtain the <u>Merkle proof</u> for this kernel.
Once they have it - they can safely reuse it. Whereas the kernel itself will eventually be deleted from the system, they still will be able to prove it was there. The proof consists of two parts:
1. Proof that a specific kernel was in the specific system state (blockchain height + hash).
1. Proof that this older state is a part of the present blockchain.


