# Beam Shaders

## Beam Smart Contracts Overview

This section will give an overview of Beam's Smart Contracts. Beam's blockchain offers fully featured Smart Contracts that support [Turing complete programming](https://www.geeksforgeeks.org/what-is-meant-by-turing-complete-in-ethereum/), enabling the construction of complex Decentralized Applications. Beam's implementation and architecture differ from most other blockchains.

### Contract Shaders and Application Shaders

In Beam, Smart Contracts have two components, Contract Shaders and Application Shaders.

{% hint style="info" %}
The term "shader" originated from 3D graphics and refers to a custom program, as opposed to a pre-defined fixed function.
{% endhint %}

Deployed on the blockchain, Contract Shaders are executed by the Beam Virtual Machine when a wallet constructs a transaction that accesses one of the public methods of the contract.

On the other hand, Application Shaders run within the Beam Wallet and act as helpers to expose the contract API to decentralized applications. One Application Shader can interac with several Contract Shaders, providing complex functionality.

<figure><img src="../.gitbook/assets/Beam Shaders.png" alt=""><figcaption><p>Conceptual Diagram of Applications and Contract Shaders</p></figcaption></figure>



Beam Shaders are implemented in WebAssembly and can be written in any programming language that can be compiled into WASM. Currently most shader are implemented in C++. Many examples of smart contracts can be found in the Beam [repository](https://github.com/BeamMW/beam/tree/master/bvm/Shaders).

## Contract Shader

Contract Shaders may do the following:

* Save/load its custom variables (access contract state)
* Lock/unlock funds.
* Create and manage assets
* Invoke public methods of other contracts
* Demand signatures for arbitrary public keys (more about this later)

### Contract life cycle

#### Creation

The contract shader is created from 2 parameters:

* Shader (the bytecode)
* Constructor argument

A valid shader must have a _Constructor_ public method, which is invoked only once, during contract creation. If the constructor is executed successfully, the new contract is created. It is assigned a unique ID which is calculated according to:

`ID := DeriveID[ Hash(Shader) | Constructor-Argument ]`

Note: Same shader may be used in different contracts, if they were created with different constructor arguments. Hence the Shader defines the contract _type_. Since the Contract ID explicitly depends on the shader and constructor arguments, it can't be tampered with.

#### Method invocation (arbitrary number of times).

Must provide the following:

* Contract ID
* Method number
* Arbitrary arguments

For the invocation the Node locates the requested contract shader, and invokes its appropriate public method with the given argument.

#### Destruction (optional)

Must provide the following:

* Destructor argument

The contract destruction is considered successful iff:

* The shader _Destructor_ method is executed successfully
* The contract deleted all its custom variables it stored.
* The BVM that tracks the external contract state ensures that
  * no locked funds left in the contract
  * no assets left created by this contract
  * there are no external references (i.e. other contracts that explicitly depend on it)

The ability of destruction is optional.

### How and when contacts are invoked

Contracts are passive, means they can only be invoked directly. There's no background processing, auto-activation on specific events, timers, or etc. This is a deliberate design decision.

To work with contracts we added 2 additional kernel types: **ContractCreate** and **ContractInvoke**.

* **ContractCreate** contains the shader of the being-created contract, and its constructor arguments
* **ContractInvoke** has the target Contract ID, its public method number, and the appropriate arguments
  * Note: contract destruction is also invoked by ContractInvoke, with the appropriate method number Contracts can only be invoked during interpretation of those kernels. Which, in turn, occurs in the context of a specific block or transaction interpretation.

We preserve the MW principle (means there're no transactions per se), means those kernels can come in a transaction in any combination, be mixed with other special kernels (CA and Lelantus-MW), and be complemented by arbitrary inputs and outputs, built by arbitrary number of users.

### Kernel validation

In MW each transaction element (input, output, and kernel) comes with a _commitment_ and validated appropriately. Originally in MW kernels's commitment may contain blinding factor only, not the value, and it's signed by the Schnorr's signature to prove this (as well as protect other kernel parameters from tampering).

Starting from HF2 we extended this concept, and introduced special kernel types (Lelantus-MW and asset control), which come with a commitment that may contain a value (which contributes to the transaction balance), and signed by appropriate means specific to the kernel type.

So we stick to the same principle with the contract invocation kernels. Their commitment, in addition to arbitrary blinding factor, should reflect the amounts that the contract invocation is supposed to lock/unlock. Their signature, in turn, is supposed to prove the following:

1. After accounting for the funds locked/unlocked by the contract, the _Commitment_ indeed consists of the blinding factor only
2. Argument of knowledge of the corresponding secret keys to all the additional public keys that the contract requested during its execution.

Technically the kernel validation goes as following:

* Kernel shader's method is executed
* During the execution the shader may:
  * Lock or unlock funds
  * Specify specific public keys `Pk[i]` that must be signed
* After the shader execution
  * Adjust the specified kernel commitment `C` w.r.t. funds locked/unlocked
    * For each locked/unlocked asset type add/subtract `H[i] * value`, whereas `H[i]` is the generator for the specified CA type.
  * For the resulting commitment `C'` the prover must prove knowledge of the appropriate blinding factor.
  * Verify the provided Multi-Signature of the argument of knowledge of the preimages of `C'` and `Pk[i]`.

Hence to build a valid transaction with the contract invocation, the user must **predict** how much **funds** the contract invocation will contribute to the transaction, as well as which **public** keys it will demand for validation.

#### Multi-signature

We use a variation of the Schnorr's signature to provide argument of knowledge of multiple secret keys. Unlike standard Schnorr's multisignature, which only proves the knowledge of the sum of the secret keys, our variant proves the knowledge of each individual secret key. This is important to mitigate the possible rogue key attack (a.k.a. key cancellation attack).

Consequently the verifier needs all the appropriate public keys in advance.

To accomplish this, we generalize the Schnorr's protocol in the following way for `M` keys:

* Prover -> Verifier
  * Context to where the signature applies (kernel parameters, etc.)
  * Set of `M` public keys `Pk[i]`
  * Public nonce `N`
* Verifier -> Prover
  * Set of `M` challenges `e[i]`
* Prover -> Verifier
  * Signature preimage `k`
* Verifier
  * Accept iff: `N + Sum(Pk[i] * e[i]) == G*k`

### Constraints and limitations

As we mentioned, shaders are executed by the BVM in a sandbox (isolated environment) to ensure that bugs or malicious behavior won't affect blockchain integrity.

#### Maximum complexity

The length of the execution (number of cycles) is limited, as well as specific support functions of the BVM have their limitations (such as max number of signatures to check, max length of the variable to store, and etc.). This is to keep execution time within sane bounds, and prevent the blockchain bloat.

For most real-world cases those limitations are adequate. If, however, for some reason the shader method is complex, and those limitations can't be met, then we recommend splitting it into several smaller ones, while the intermediate calculation result is saved in some custom variables.

#### Repeatability

In addition it's critically important to ensure that the shader executes in an exactly the same way on all the nodes, and produces exactly the same side effects (if not - attacker can easily cause chain split). Because of this:

* All the memory that the shader can access is initialized the same way on all the nodes.
  * Developers however should not assume that it's necessarily zero-initialized.
* By design, there are no BVM functions for the shader that can yield different results. Such as generating randoms, getting current time, and so on.
* **Native floating-point operations are currently not supported**.
  * This is because there may be subtle differences in FPU (floating-point unit) behavior on different machines (like the positive/negative sign of zero). In the future we may support it, once all the potential problems are solved.
  * Instead we recommend using multi-precision integer arithmetics.

#### Contract bounds

Contract can access its custom variables only. It can't access the variables that belong to other contract, not even for reading. This is a deliberate design decision. By such it's possible for the contract to protect its data from unauthorized usage by other contracts (though that data itself is visible to all the users).

#### Other considerations

We detect and protect against malicious behavior toward the blockchain integrity (such as invalid memory access, attempt to unlock more funds that had been locked, attempt to manage an asset that doesn't belong to the contract, and so on).

With all the precautions from our side, there is **literally no guarantee that contract behaves as described by its creator**. Innocently-looking code may have bugs or disguised backdoors. This is the price of the flexibility, an inevitable trade-off of the customizable logic.

Hence users should only trust contracts after thorough source code audit. The compilation process should be transparent, i.e. everyone should be able to take the source code, compile it with publicly-available compiler, and get exactly the same shader bytecode.

## Wallet-side shaders

As we mentioned, wallet-side shaders are designed for wallets (user-side software), to provide an interface to specific contract types. In contrast to contract-side shaders, the wallet-side shaders don't have strict complexity limitations, and are allowed to read any blockchain information (state and variables of any contract, block headers). There is also no consideration regarding repeatability, so they can access current time, generate random, and so on.

We make sure however that they can't do anything potentially dangerous without user authorization.

* Shader can get public keys generated by the user account, but not private keys.
* It can ask to communicate with other users (via SBBS system), this may be necessary for multi-user signatures in specific contracts
  * Communication must be allowed by the user explicitly.
* Shader may prepare contract control kernel and ask the wallet to build and broadcast the appropriate transaction
  * Of course this requires user authorization. The user sees how much funds it gets/spends in the transaction.

Technically the wallet-side shaders is executed with user-supplied parameters (in a textual form), and can do either of the following:

* produce json-style document with the relevant information for the user
* prepare contract control kernel, that, after user authorization, is signed by the wallet and used in the appropriate transaction.
