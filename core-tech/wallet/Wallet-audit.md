Crypto-currencies are often criticized for being used by "gray market" and virtually inappropriate for compliant businesses due to their inherent non-transparency.

So we'd like to give compliant businesses appropriate tools to be as transparent as possible for the appropriate authorities, while remaining confident to others. We'll present a scheme how it can be implemented in BEAM, and review it.

# High-level design

The wallet used for compliant business (we'll call it a business wallet) should be configured for auditing. It generates a public/private key pair specifically for audit. The public key is given to the appropriate auditor(s). Optionally the business wallet may generate several public/private auditor key pairs for different authorities.

Note: The "public key" is actually <u>confidential</u>, and should not be disclosed by the auditor to others to keep the anonymity of the business.

Every transaction that the business wallet performs is _tagged_, such that only the auditor may identify it in the blockchain using the provided public key. The auditor however is unable to create such a _tagged_ transaction using the public key alone.
By tagging the transaction the <u>business wallet commits to disclose (off-chain) all the relevant info for the auditor, when required to</u>. After receiving all the information the auditor verifies that it indeed conforms to the original commitment.

The original transaction details the auditor may want are:

* Input/Output UTXOs of this business wallet.
   * Note: The full transaction contains also the Input/Output UTXOs of other wallets. But those should not be disclosed (unless absolutely required) to keep their anonymity.
* Proof of the ownership and amount of every input/output UTXO which belongs to this business wallet, without disclosing the blinding factors (public signature).
* Optionally any additional information: digital documents, signatures, arbitrary text message, etc.

The task of the auditor is the following:

* Run the full Node, which (in particular) downloads the original blocks (this is important!)
* Find all the tagged transactions for all the business wallets it's aware of.
* Request all the transaction information (off-chain).
* After receiving the information the following is verified:
   * All the transaction components are valid (signatures and etc.)
   * The transaction details indeed correspond to the original commitment
   * All the referenced inputs and outputs indeed present in the original block.
   * **Verify the completeness of the transaction graph of this business wallet.**

By such a scheme the business wallet is enforced to report about the transaction in advance, and to commit to all the details that will be revealed to the auditor later.

The auditor is able to reconstruct the whole business ledger. It should build the transaction graph and verify its completeness, which means that every UTXO that the business wallet spends must have been reported earlier by the same business wallet upon receiving. By such the business wallet will be able to spend legally only what it received legally. So that, unlike traditional ledgers, the auditor can see the life cycle of every UTXO, and not only of the overall balance. So that the "_Pecunia non olet_" is somewhat less obvious.

# Technical Design.

All transactions include transaction kernels, and we'll use this for transaction tagging. Each kernel, even "minimal", without any optional data, already has two degree of freedom:
1. Excess (total blinding factor)
1. A _nonce_, which is a part of the Schnorr's signature

By utilizing those two degrees of freedom it's possible to achieve the two needed effects:
* Tag the transaction such that it can be identified using the auditor's public key.
* Embed the commitment to the transaction details

For every transaction the business wallet will add an extra kernel for the auditor. Moreover, if it's auditable by different authorities who gets different public keys - the business wallet will add such a kernel for every key (in every transaction).

Note: If the other transaction party is also a business wallet, which is obliged to report to appropriate authorities - it will add its kernel(s) to the transaction as well. There is no collision here (transactions may contain arbitrary number of kernels).

First the business wallet collects all the info that it's going to reveal to the auditor, saves it, and computes its Hash. This is the commitment to the transaction details. The kernel _Excess_ is derived from this value.

Now, to flag the transaction the business wallet signs the kernel in a special way. The _Nonce_, which is a part if the Schnorr's signature, is derived from the visible _Excess_ and the auditor's public key, hence the auditor can identify such a kernel. Yet the auditor can't craft such a kernel using only its public key, because in order to create Schnorr's signature it's essential to know the _Private Nonce_ value (before it's multiplied by G-generator).

## Flow diagram:

### Business Wallet, transaction building:

* `TxDetails` = { my Inputs + public signatures, my Outputs + public signatures, Some other arbitrary data (docs and etc.) }
* Save `TxDetails` (locally)
* For each auditor key:
   * `Nonce1 = HMac(AuditorKey.Private, TxDetails)`
   * `PrivateKey = Nonce1 * Hash(G * Nonce1 | TxDetails)`
   * `Kernel.Excess = G * PrivateKey`
   * `Nonce2 = Hash(Kernel.Excess | AuditorKey.Public) * AuditorKey.Private`
   * `Kernel.Signature.Nonce = G * Nonce2`
   * Sign kernel using
      * key = `PrivateKey`
      * Nonce = `Nonce2`
   * Add the kernel to the transaction
   * `Transaction.Offset -= BlindingFactor`

### Auditor

* Download blocks (as usually Nodes do)
* For every new block, for every kernel, for every known public key
   * `ExpectedNonce = Hash(Kernel.Excess | AuditorKey.Public) * AuditorKey.Public`
   * If (`Kernel.Signature.Nonce == ExpectedNonce`)
      * Save the info: Appropriate business wallet, Block ID (Height + Hash), and the `Kernel.Excess`
      * Request the transaction info for this kernel, as well as `G * Nonce1`
* After receiving the transaction info for the specified kernel
   * Verify that the `TxDetails` is sane (correct format, signatures, etc.)
   * Verify that the presented data corresponds to the original commitment:
      * `Kernel.Excess == (G * Nonce1) * Hash(G * Nonce1 | TxDetails)`
   * Verify that the asserted inputs and outputs indeed present in the original block.
   * **Verify that all the referenced inputs were indeed reported earlier**
   * Mark inputs as consumed, realize the outputs.

### Notes:

Note: the blockchain may sometimes "rollback", and this should be accounted for by the auditor. There are basically two options:

1. Complex solution: the auditor must know to undo the changes to the affected ledgers. It may request the transaction details for different history branches, and implement navigation, which involves undo and redo to all the affected ledgers.
1. Simple solution, which may be practically good enough. There is a maximum threshold for the rollback range, so it is sufficient just to wait for this period before requesting/processing the transaction details. And the auditor may assume that larger rollbacks are impossible, as long as the whole system operates normally.

Minor note: in a highly unlikely case (with probability order of 2<sup>-256</sup>) there may be a false positive when chasing for the auditor kernels, i.e. a kernel that looks like it's an auditor kernel, but it's actually not. Though highly unlikely, perhaps there should be a possibility for the business wallet to deny this transaction? On the other hand false negatives are not possible. I.e. no excuses if the transaction was not flagged in advance.

### Soundness of the scheme

First we'll prove the soundness of the commitment to the transaction details, i.e. that the business wallet will have to reveal the original transaction details when requested.

The auditor verifies the following:
      * `Kernel.Excess == (G * Nonce1) * Hash(G * Nonce1 | TxDetails)`

So, in order to substitute a different `TxDetails` the Wallet will have to find a different EC  point `G * Nonce1` that satisfies the above equation. Given there's no direct relation between the hash function, and the EC point multiplication, it seems to be the problem of finding collisions of a composite one-way function, which should not be feasible.

Next we prove that it's impossible (unless with negligible probability) to create such a kernel without the knowledge of the `AuditorKey.Private`.

The auditor checks the following:
   * `Kernel.Signature.Nonce == Hash(Kernel.Excess | AuditorKey.Public) * AuditorKey.Public`
   * Schnorr's signature

In order to create Schnorr's signature it's essential to know both the private key, and the preimage of `Kernel.Signature.Nonce` (we'll skip the argument here), wherease the preimage is given by `Hash(Kernel.Excess) * AuditorKey.Private`. Since the `Kernel.Excess` and `AuditorKey.Public` are known, the auditor private key should be known as well.

### Zero-knowledge of the scheme

First we'll prove that this kernel is indistinguishable unless `AuditorKey.Public` is known. The following is verified:
   * `Kernel.Signature.Nonce == Hash(Kernel.Excess | AuditorKey.Public) * AuditorKey.Public`

Note that we intentionally substituted the `AuditorKey.Public` into the hash function. If that was not the case, the attacker could calculate its value from the visible `Kernel.Excess`, and then notice the linear dependence of the `Nonce` by this value (for several kernels tagged for the same auditor).
But since `AuditorKey.Public` is included in the hash, its value is obscured, and the `Nonce` looks like a random EC point.

Let's prove that the auditor private key is not leaked. The auditor verifies the Schnorr's signature. Means:
* `G * Preimage = Hash(<some known value>) * AuditorKey.Public + <challenge> * <known EC point> * Hash(<known EC point>)`

Which can be rephrased as:
* `G * AuditorKey.Private = G * Preimage * <some value> - <known EC point> = <some EC point>`

So, it's a pure discrete logarithm problem.

# Review of this scheme

The good:
* Strict demand to reveal the fact of the transaction in advance, and commitment to the original transaction details.
* The fact of the transaction taking place and the report about the transaction is the same, because the transaction is atomic!
   * Means, there is no way the transaction is reported but didn't actually take place, blockchain rolled back, or whatever other reason, or vice versa.
* Inability to use "gray" money in compliant transactions. All the inputs must have been reported earlier.
* The auditor can fully reconstruct the transaction graph, but it can discover nothing about other transaction parties.
   * This is very valuable feature (perhaps the most valuable).
* The auditor can actually see when the business wallet spends its UTXOs, even if it doesn't flag the transaction.
   * For this the auditor will have to analyze all the UTXOs being spent in the original block, which is feasible.
   * Naturally the opposite isn't true: it's possible to receive UTXO without flagging the transaction (but it won't be possible to spend it in a compliant transaction).

The bad:

* Naturally there is no way to verify there is no "gray" ledger of the same business, it may have other wallets, or conceal part of the transactions.
   * This is also not solvable in "traditional" world.
   * However we guarantee the separation between "compliant" transactions and the others.
* No way to verify that the original transaction actually included the reported inputs/outputs, as well as nothing beyond them.
   * We do verify that they were present in the original block, and that the reporter actually owns them (by demanding signatures), but they may be parts of another transactions.
   * This is equivalent (in some sense) to a situation where you report about transaction, but pay to others, or show less money than is actually transferred.
   * This is also unsolvable in the "traditional" world.
