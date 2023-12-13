---
description: >-
  This page describes CA(Confidential Assets), Lelantus-MW, as well as will
  enable adding more features in the future.
---

# Implementation of Confidential Assets

Implementing confidential assets on the Beam blockchain takes advantage of the LelantusMW protocol enhancing the privacy and security for all transactions.

* **CA support**
  * Blinded asset tags, similar to the Elements design by A. Poelstra, can optionally be associated with each Unspent Transaction Output (UTXO).
  * Asset tags have a proof of validity based on the 1-out-of-many Sigma protocol, developed by [Jens Groth](https://eprint.iacr.org/2014/764.pdf).
* **Shielded pool (e.g., LelantusMW)**
  * CA support for shielded operations.
  * One-side payments and direct anonymous payments support.
* The system design is heterogeneous in nature:
  * All kernels carry excess blinding factors and may include extra validation rules such as height lock and relative lock.
  * Some kernels may control subsystems:
    * Asset control (creation, emission).
    * Shielded operations (mint, spend).
  * Kernels not only affect the balance but include side effects.

## Confidential assets support


This current design differs significantly from the previous, older design.


We identify each asset by `AssetID` as a 32-bit integer. For each asset there's an appropriate NUMS generator, which is generates deterministically from the `AssetID` (via hashing). The `AssetID == 0` is reserved for default asset (Beam).

### UTXO encoding

Due Mimblewimble (MW), it is feasible to encode UTXOs (asset types) using different NUMS (nothing-up-my-sleeve) generators. The UTXO representing the asset is comprises the following:

* Blinded generator: $$H^* = H_i + k_A•G$$
* Asset surjection proof (verifies the provided generator is indeed one of the generators listed (with arbitrary blinding factor added).
* Pedersen commitment: $$C = k•G + v•H^*$$
* Rangeproof (bulletproof): in terms of this blinded generator.

#### Asset surjection proof

Derived from the Sigma protocol, where the prover specifies a range of `AssetID` values, and proves that the specified generator with arbitrary blinded factor is one of the generators within that range.

Meanwhile, the verifier generates a list of asset generators for the provided range, methodically subtracting the provided blinded generator from each element on the list. The Prover confirms the Sigma protocol by providing the opening of one of the elements for the blinding factor, i.e., the `G` generator.

### Asset control

Asset controls grants any user permission to create new asset types vs Beam which are automatically released into circulation with each new block generated. Assets are emitted and burned by the asset owner. The asset lifecycle has three stages: Asset creation, asset emission/burn and asset destroying.

#### Asset creation

When sending an asset creating transaction type, it provides both the **Owner key** and associated **metadata**. Any proceeding asset actions will require a private key signature, and metadata visible to all users is immutable once the asset is created.


Metadata reserves a significant number of Beams when locking an asset, meaning that this transaction implicitly uses up that amount. If the transaction is successful, the system assigns the lowest available and unused `AssetID` to the asset and links it to the asset.


#### Asset emission/burn

The user initiates a transaction by sending a unique asset emission kernel, which can have a positive or negative emission amount. To complete the transaction, the kernel excess blinding factor must be signed by the appropriate private key of the asset owner.

This transaction will automatically create or consume a certain amount of the asset, which should be balanced out by other transaction elements such as inputs and outputs.

#### Asset destroying

To destroy an asset, an asset-destroying kernel with owner key signature is required. Once the asset has been destroyed, the `AssetID` is no longer linked to the owner, and the locked Beams  return to the user.


Asset can be destroyed only if:

* Total burn occurs.
* Minimum lock period elapsed after asset burn completes without an emitted event.


This minimum lock period prevents any asset owner manipulation or tampering. For example, if a user requests a specific asset, the `AssetID` must be directly linked with the asset info (Metadata and owner key) before its transaction expires. This tweak prevents the asset owner from destroying or re-create an asset during the lock period.

### Asset state

The current state of the system includes a commitment to the latest state of assets, represented as an MMR root of all active assets and their relevant information:

* Static info (Metadata and owner public key)
* Current emission value
* Lockheight:  The system maintains the latest height of the asset's transition from being burned to not being burned. Based on this information, users can take certain actions:
  * Asset owner can determine whether and when the asset can be destroyed, and consequently unlock the funds that were previously locked.
  * Other users can estimate the minimum height range within which the asset can be safely used. This helps determine whether the asset was subject to reorgs in the past or could be tampered with in the future.


Locking funds for asset creation is necessary to prevent spamming in the system. If there are too many assets, it can cause the Nodes to become heavier and can also reduce the effectiveness of the asset surjection proof due to a limited anonymity set. Although a significant amount of funds need to be locked, this design should not be an issue for users who are experimenting with assets as they will receive their funds back upon completion.

However, for organizations selling assets to users, it is unclear if they will ever be able to burn their assets back, as they must first own all of their assets. Despite this, the risk seems justified.


### Note on transaction repeatability

UTXO-based systems, including MW, face a unique issue of repeatability. When an attacker possesses all the inputs of a transaction where they pay someone (which is typically the case), they can repeat this transaction in the future without the recipient's consent. Furthermore, if the recipient spends only the inputs received from the attacker, the attacker can repeat those transactions as well, and so on. This process can continue until a certain depth is reached where additional inputs are required, which were not initially obtained from the attacker.

Trading only beams reduces the impact of 'replays' since users can only receive beams, not lose them. However, the situation is different when it comes to asset-controlling transactions. Since assets are created through transactions, attackers can potentially replay these transactions and cause additional asset emissions, leading to a significant problem. Additionally, by replaying these transactions, attackers may be able to gain some of the newly created assets, further compounding the issue.

To mitigate this threat, starting from Fork2, duplicating kernels will be forbidden. Technically this is achieved by the following:

* Additional consensus parameter (`MaxKernelLifespan)` equivalent to roughly \~ one month.
* Starting from Fork2, kernels with `HeightLock.Min` (minimum height) lower than Fork2 will be rejected.
* Each kernel, in addition to the optional `HeightLock.Max` (maximum height) will have an implicit max height lock as `HeightLock.Min + MaxKernelLifespan`. This (and the previous restriction) will make repeating old kernels impossible.
* Each node will track all recent kernels down to the current height minus `MaxKernelLifespan`. Kernels that are below this height may be removed from the system for the purpose of blockchain verification.
* Side effect: Kernels with relative height lock (available on mainnet) will not be able to reference a kernel older than `MaxKernelLifespan`. However, this should not pose an issue as relative locks require much shorter duration. This effectively makes kernel replaying illegal, reducing the burden on the nodes to track only the most recent kernels.

## Shielded pool (a.k.a. Lelantus-MW)

**Disclaimer**: The [Lelantus Protocol](https://lelantus.io/) is the creation of Zcoin's cryptographer Aram Jivanyan as part of research aimed at improving privacy protocols. Our team's design and implementation are based on the publicly-available Lelantus scientific paper. All our code was developed from scratch based on this paper as our reference.

While Lelantus greatly improves upon privacy, to solve the issue of MW linkability, users can recycle funds via a shielded pool. This tweak from the original Lelantus protocol is unique in the following ways:

* Transaction values are never revealed.
* The Lelantus Protocol formulates transactions as mint/spend primitives, which can be combined with MW and shielded parts in any way that satisfies the balance-to-zero principle used in MW.
* Native CA support.

In technical terms, the system supports the following in addition to standard transaction elements:

* Shielded output: transfers a portion of MW into shielded pool
* Shielded input: withdraws a portion of the shielded pool back into MW (both are deemed as special transaction kernels on the blockchain.

To supplement the standard MW blinding factor generator `G`, an extra generator `J` is available for the secondary blinding factor, i.e., the _serial number_.

### Shielded output

Consists of the following:

* Blinded serial number commitment: $$C_s = k_s•G + s•J$$
* Generalized Schnorr's signature that proves the above commitment is indeed of this form
* Optionally asset info: the blinded asset generator + asset surjection proof.
* UTXO commitment $$C_{MW} = k_{MW}•G + v•H$$
* Rangeproof

The UTXO commitment $$C_{MW}$$ (without the serial number) is accounted for when verifying the transaction balance. After verification, rather than combining UTXO set, the following double-blinded commitment goes into the shielded pool:

* $$C = C_s + C_{MW} = s•J + (k_s + k_{MW})•G + v•H$$

The shielded outputs in the pool form a sequence of commitments (EC points).


The serial number (**s**) is obtained from a different public key (`SpendKey)` which discloses during the spending process. In addition, the prover will need to prove the knowledge of the appropriate private key.

In addition, the $$C_s$$ commitment must be unique. This prevents accidental misuse, and avoids making subsequent element withdrawal impossible.


### Shielded input

Consists of the following:

* Range within the shielded poo containing the being-spent element.
* `SpendKey` is disclosed, providing the key signature needed for the shielded input.
* Optionally asset info: the blinded asset generator + asset surjection proof.
* Output commitment $$C_{out} = k_{out}•G + v•H$$
  * It should commit to the same value, but the blinding factor $$k_{out}$$ differs from that used in shielded output.
* Generalized Schnorr's signature, that proves the $$C_{out}$$ is indeed of this form.
* Sigma proof for the rest.

The `SpendKey` must be unique to prevent double-spending of an asset.

During verification, the verifier computes the serial number (**`s`**) from the `SpendKey` while calculating the following:

* $$C = C_{out} + s•J$$ This EC point is subtracted (methodically) from all the elements in the referenced range of the shielded pool.

If everything is correct, then the element spent transforms into:

* $$C = (k_s + k_{MW} - k_{out})•G$$ Note that both as the set and serial number generators `H` and `J` are eliminated. The prover proceeds to demonstrate their knowledge of the opening of one of the elements within the range, using only the `G` generator.

### One-side payments, and direct anonymous payments

The shielded pool enables one-side payments (MW transactions require mutual agreement from both parties). This is because the serial number (**S**) is derived from an arbitrary public key `SpendKey`, which can be calculated by the sender without the disclosure of the appropriate private key.

However, this method is not totally anonymous as the sender can see when the receiver spends the funds. To solve this issue, the shielded output consists of two parts: the $$C_s$$ and $$C_{MW}$$. During the initial setup, the receiver generates and sends an arbitrary number of different $$C_s$$ elements (with their Schnorr's signatures). The sender then uses these elements in the shielded output without knowledge of the serial number.

Our scheme enables the receiver to scan the blockchain to detect all its shielded outputs without needing an auxiliary channel. For $$C_s$$ all the owner info is embedded within the Schnorr's signature (which has a degree of freedom). For the $$C_{MW}$$, the owner info is embedded in the Schnorr's signature, while the requires info recovery from the bulletproof.

Ultimately, the following information is recovered:

* All the relevant parameters: blinding factor, `SpendKey`, value, `AssetID`
* Is it visible to the sender, i.e., was the $$C_s$$ created by the sender or the receiver in advance.
* Sender ID (a public key belonging to the sender).
* Default 32-byte message.


The aforementioned information can be retrieved through the `Owner` key. However, the `master` key is still necessary to spend it. This enables the utilization of the owner key in owned nodes to recognize owned TXOs and shielded elements without jeopardizing the funds in the event of a node compromise.


### Implications and constraints

The Lelantus is a great technology, **but it comes with known limitations**.

* Scalability (size).
  * No MW cut-through for the shielded inputs/outputs. MW cut-through applies to non-shielded inputs/outputs.
    * Shielded output \~800 bytes.
    * Shielded input \~1.6KB, can vary depending on anonymity set size.
    * If asset type is blinded: two more asset proofs (output and input), another \~2KB
* Verification time.
  * Roughly one second for 64KB elements (very big).
  * Easily parallelized
  * Only 10 milliseconds for each additional proof for the same anonymity set (batch verification).
  * During initial sync, many blocks can be batch-verified at once.

Thus, in order to create a sensible system that reaps the advantages of MW while also disrupting linkability, we have developed the following design:

* The majority of transactions should still use MW.
* The number of shielded inputs/outputs in a block is limited, creating a competitive fee market.
* The spend window, which determines the anonymity set size, is limited, and this limit is further reduced if the element being spent is not one of the most recent.

The maximum spend window or anonymity set size is yet to be decided but will likely be between 50,000 to 100,000 KB. The number of shielded elements in a block will be limited to create this window over several days.

Additionally, users can only spend their shielded elements with the maximum spend window if it references the most recent elements. If they miss their opportunity, they will have to spend it in a smaller spend window of around 1,000 elements. However, they can recycle it through the shielded pool again.

As a result, our team has designed a network that offers equal amounts of **scalability and performance**:

* Not too many elements that can't be cut-through
* Reasonable verification times: shielded inputs will have large overlap.

But importantly, those restrictions will also lead to **better privacy**. Here's why:

### Privacy

In order to understand how effectively privacy is achieved through hiding within a crowd, it's important to define two terms: absolute and relative anonymity sets.

* The absolute anonymity set size refers to the total number of individuals in the set that the user has chosen to hide among.
* The relative anonymity set size, on the other hand, is the ratio of the chosen absolute set size to the overall set size weighted by the probabilities of hiding in each potential subset.

In simpler terms, the relative set size represents the probability that a user will choose a specific absolute set. To achieve a high level of privacy, both the absolute and relative sets should be maximized. If the absolute set size is small, the user may already be suspected. If the relative set size is small, the user can still be de-anonymized through repeated transactions, even if the absolute anonymity set is large. [A good explanation by Ian Miers is here](https://zfnd.org/blockchain-privacy-equal-parts-theory-and-practice/).

Due to the finite size of the anonymity set in Lelantus, a compromise must be made.

* If too few users use the system, then each user's activity can be easily identified.
* However, if too many users use the system, the anonymity set is filled up quickly, resulting in a smaller relative set size and a lower probability of an unrelated user falling into the same set.

In contrast, systems with an unlimited anonymity set size, such as Zcash, have an advantage in this regard. However, in practice, the difference may not be significant since users typically spend their recent outputs, which can be assumed by attackers with a significant probability even in Lelantus.

To accurately estimate the practical privacy of the system, real-world usage data may be needed.
