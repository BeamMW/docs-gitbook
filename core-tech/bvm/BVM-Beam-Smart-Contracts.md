# Overview of the Beam contracts ecosystem

## Motivation

Beam was initially based on MW protocol. Starting from HF2 we extended it with CA (confidential assets) and Lelantus-MW to address the linkability problem. With all those extensions Beam still operates according to the well-defined <u>fixed</u> logic.

There're so-called _scriptless scripts_, which means that in particular cases it's possible to emulate script behavior off-chain, and broadcast only the end-result to the network. However the variety of such cases is limited to things like aggregate signatures, atomic swaps, payment channels and etc.

To overcome those restrictions we decided to allow for programmable (custom) logic executed on-chain.

## Terminology
### Shader
The term shader comes from a 3D graphics. It means a custom program, as opposed to the pre-defined fixed function.

In Beam Shader is a program designed to run within the BVM (beam virtual machine), and implement the custom contract logic. It's executed in an isolated secure environment (a.k.a. sandbox). In addition to general-purpose computations, shaders may call support functions of the BVM, to access things outside its sandbox in a controlled way.

Shaders should have _public_ methods, which can be invoked either directly from a block/transaction, or another contract.

### Contract
Contract is a logical object, managed by the node, and operated by its shader. Once the contract is created, its shader can **not** be changed, Means the contract logic is guaranteed to remain constant.
The contract may do the following:
* Save/load its custom variables (access contract state)
* Lock/unlock funds.
* Create and manage assets
* Invoke public methods of other contracts
* Demand signatures for arbitrary public keys (more about this later)

## Components of the system

To create a new contract the developers should implement the appropriate shader. Currently Beam supports wasm (WebAssembly). So _theoretically_ shader can be written in any programming language that can be compiled into wasm.
We recommend using LLVM to compile into wasm. Currently our examples are built using LLVM (written in C++ and compiled using clang).

In addition to the shader that implements the contract logic, the wallet should be able to realize it, provide the user with appropriate UX (such as a dashboard), and be able to build appropriate transactions that operate the contract.

We decided to implement this via shaders too. So that there are 2 kinds of shaders: node-side shaders that operate the contract, and wallet-side shaders, that provide the appropriate functionality in the user wallet. By such anyone can create new contracts and integrate them into the wallet, without the need to modify the original wallet code.

# Node-side

## Contract life cycle

### Creation
The contract is created from 2 parameters:
* Shader (the bytecode)
* Constructor argument

A valid shader must have a _Constructor_ public method, which is invoked only once, during contract creation. If the constructor is executed successfully, the new contract is created. It is assigned a unique ID which is calculated according to:

`ID := DeriveID[ Hash(Shader) | Constructor-Argument ].`

**Note**: Same shader may be used in different contracts, if they were created with different constructor arguments. Hence the Shader defines the contract _type_.
Since the Contract ID explicitly depends on the shader and contructor arguments, it can't be tampered with.

### Method invocation (arbitrary number of times).
Must provide the following:
* Contract ID
* Method number
* arbitrary arguments

For the invocation the Node locates the requested contract shader, and invokes its appropriate public method with the given argument.

### Destruction (optional)
Must provide the following:
* Destructor argument.
		
The contract destruction is considered successful iff:
* The shader _Destructor_ method is executed successfully
* The contract deleted all its custom variables it stored.
* The BVM that tracks the external contract state ensures that
  * no locked funds left in the contract
  * no assets left created by this contract
  * there are no external references (i.e. other contracts that explicitly depend on it)

The ability of destruction is optional.

## How and when contacts are invoked

Contracts are **passive**, means they can only be invoked directly. There's no background processing, auto-activation on specific events, timers, or etc. This is a deliberate design decision.

To work with contracts we added 2 additional kernel types: _ContractCreate_ and _ContractInvoke_.
* _ContractCreate_ contains the shader of the being-created contract, and its constructor arguments
* _ContractInvoke_ has the target Contract ID, its public method number, and the appropriate arguments
  * Note: contract destruction is also invoked by ContractInvoke, with the appropriate method number

Contracts can only be invoked during interpretation of those kernels. Which, in turn, occurs in the context of a specific block or transaction interpretation.

We preserve the MW principle (means there're no transactions per se), means those kernels can come in a transaction in any combination, be mixed with other special kernels (CA and Lelantus-MW), and be complemented by arbitrary inputs and outputs, built by arbitrary number of users.

## Kernel validation

In MW each transaction element (input, output, and kernel) comes with a _commitment_ and validated appropriately. Originally in MW kernels's commitment may contain blinding factor only, not the value, and it's signed by the Schnorr's signature to prove this (as well as protect other kernel parameters from tampering).

Starting from HF2 we extended this concept, and introduced special kernel types (Lelantus-MW and asset control), which come with a commitment that may contain a value (which contributes to the transaction balance), and signed by appropriate means specific to the kernel type.

So we stick to the same principle with the contract invocation kernels. Their commitment, in addition to arbitrary blinding factor, should reflect the amounts that the contract invocation is supposed to lock/unlock.
Their signature, in turn, is supposed to prove the following:
1. After accounting for the funds locked/unlocked by the contract, the _Commtiment_ indeed consists of the blinding factor only
1. Argument of knowledge of the corresponding secret keys to all the additional public keys that the contract requested during its execution.

Technically the kernel validation goes as following:
* Kernel shader's method is executed
* During the execution the shader may:
  * Lock or unlock funds
  * Specify specific public keys `Pk[i]` that must be signed
* After the shader execution
  * Adjust the specified kernel commitment `C` w.r.t. funds locked/unlocked
     * For each locked/unlocked asset type add/subtract `H[i] * value`, whereas `H[i]` is the generator for the specified CA type.
  * For the resulting commitment `C'` the prover must prove knowledge of the appropriate blinding factor.
  * Verify the provided _Multi-Signature_ of the argument of knowledge of the preimages of `C'` and `Pk[i]`.

Hence to build a valid transaction with the contract invocation, the user must <u>predict</u> how much <u>funds</u> the contract invocation will contribute to the transaction, as well as which <u>public keys</u> it will demand for validation.


### Multi-signature

We use a variation of the Schnorr's signature to provide argument of knowledge of multiple secret keys. Unlike standard Schnorr's multisignature, which only proves the knowledge of the sum of the secret keys, our variant proves the knowledge of each individual secret key. This is important to mitigate the possible rogue key attack (a.k.a. key cancellation attack).

Consequently the verifier needs all the appropriate public keys in advance.

To acompish this, we generalize the Schnorr's protocol in the following way for `M` keys:
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

## Constraints and limitations

As we mentioned, shaders are executed by the BVM in a sandbox (isolated environment) to ensure that bugs or malicious behavior won't affect blockchain integrity.

### Maximum complexity
The length of the execution (number of cycles) is limited, as well as specific support functions of the BVM have their limitations (such as max number of signatures to check, max length of the variable to store, and etc.). This is to keep execution time within sane bounds, and prevent the blockchain bloat.

For most real-world cases those limitations are adequate.
If, however, for some reason the shader method is complex, and those limitations can't be met, then we recommend splitting it into several smaller ones, while the intermediate calculation result is saved in some custom variables.

### Repeatability
In addition it's **critically** important to ensure that the shader executes in an exactly the same way on all the nodes, and produces exactly the same side effects (if not - attacker can easily cause chain split). Because of this:
* All the memory that the shader can access is initialized the same way on all the nodes.
   * Developers however should **not** assume that it's necessarily zero-initialized.
* By design, there are **no** BVM functions for the shader that can yield different results. Such as generating randoms, getting current time, and so on.
* <u>Native floating-point operations are currently not supported.</u>
   * This is because there may be subtle differences in FPU (floating-point unit) behavior on different machines (like the positive/negative sign of zero). In the future we may support it, once all the potential problems are solved.
   * Instead we recommend using multi-precision integer arithmetics.

### Contract bounds

Contract can access its custom variables only. It can't access the variables that belong to other contract, not even for reading. This is a deliberate design decision. By such it's possible for the contract to protect its data from unauthorized usage by other contracts (though that data itself is visible to all the users).

### Other considerations

We detect and protect against malicious behavior toward the blockchain integrity (such as invalid memory access, attempt to unlock more funds that had been locked, attempt to manage an asset that doesn't belong to the contract, and so on).

With all the precautions from our side, there is <u>literally no guarantee that contract behaves as described by its creator</u>. Innocently-looking code may have bugs or disguised backdoors. This is the price of the flexibility, an inevitable trade-off of the customizable logic.

Hence users should only trust contracts after thorough source code audit. The compilation process should be _transparent_, i.e. everyone should be able to take the source code, compile it with publicly-available compiler, and get exactly the same shader bytecode.

# Wallet-side shaders

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
