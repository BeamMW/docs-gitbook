# Beam Platform Architecture

## Confidential Assets

Beam Blockchain natively supports creating new types of tokens called Confidential Assets (CA). Confidential Assets inherit all the properties of the Beam Coin (a native token on Beam Blockchain), namely, confidentiality and scalability using MimbleWimble's cut-through process.

Unlike Ethereum ERC-20 contracts, Beam assets are implemented on Layer 1 and are native citizens of Beam's ecosystem. Each Beam transaction can simultaneously include various CA types as inputs and outputs, provided they match the CA type sum to zero, i.e., no type conversion can occur within a transaction.

## Transaction Kernels

Beam transactions are comprised of inputs and outputs, which are represented by Pedersen Commitments with adjacent zero-knowledge range proofs to ensure committed values are positive within a specific range. While inputs and outputs can be removed during the cut-through process, transaction kernels remain in the blockchain and are crucial for protocol validation.

Initially, kernels contained the signed difference between the blinding factors but were extended to support various purposes during Beam's development, including reflecting transaction type and supporting Lelantus shielded pool. Additionally, the kernels were further repurposed to enable the deployment of smart contracts on the Beam blockchain

## Kernels with Side Effects

A transaction kernel can identify any side effects, i.e., direct changes to the node state, including transformations to tx inputs and outputs. When this occurs, Beam coins transform into a hybrid UTXO based <> State Based cryptocurrency. When a side effect modifies the outputs, the value stored records on the blockchain as the kernel "consumed" the output. This results in the kernel "emitting" new inputs using the value stored in the node state.

## Beam Shaders and Beam Virtual Machine

The encapsulation of each node state is done using Beam Shaders while defining the state variables and correlating business logic to modify them. These shaders are coded using WebAssembly (WASM), allowing seamless deployment on Beam Virtual Machines.

Beam uses WASM's binary instruction format for stack-based virtual machines and takes advantage of WASM's industry-standard support for various web for client and server applications.

## Using Beam Shaders to Implement Smart Contracts

For a more technical description of Beam Smart Contracts, read here

According to [Investopedia](https://www.investopedia.com/terms/s/smart-contracts.asp), Smart Contracts are _"a self-executing contract with the terms of the agreement between buyer and seller being directly written into lines of code"_ Smart Contracts on the Beam blockchain are executed via Beam Shaders.

Each Beam Shader is a collection of state variables and methods, including a constructor and a destructor. The **shader constructer** is only executed once when the Contract Creator defines parameters when registering a new shader. When a new Beam Shader code and construction parameters records on the blockchain, it creates a unique Shader Instance.

It is important to note the same code with different parameters will produce a distinct unique Shader ID, thus creating a separate Shader Instance.

A **destructor** runs when the Beam Shader is deactivated and cleans all used resources.  A Beam Shader can only be deactivated if it does not contain any funds, preventing the destruction of value during the Shader deactivation process.

## Properties of Chain Side Beam Shaders

The section below details key properties of smart contract (Beam Chain Side Shaders) implementation on the Beam blockchain.

* Beam Shaders utilize several limiting mechanisms (charging execution fees, maximum execution fees) to avoid blocking the node with an unbounded cycle and ensure bounded execution time.
* **Shaders cannot be automated** and only record on the blockchain via sent transactions with required parameters, signatures, and fees.
* Shaders can claim a portion of fees from provided sources of decentralized application requests. Shaders can then store and issue disbursements, including refunding excess payments to special [Vault contracts](https://blog.giveth.io/the-vault-contract-open-sourced-by-giveth-fe2261f7b91b).
* Shaders can communicate with other Shaders to delegate operations in the transaction context, activating the initial Shader on the chain.
* The Shader that controls the assets can burn CA emissions, and the shaders defined logic conducts emission burning when manual emission burn cannot function.
* When a Shader deploys, it results in one of two outcomes: either the Shader consumes funds (transaction inputs) or unlocks funds (transaction outputs) recorded on the blockchain.
* **Overdraft protection**. BVM only validates equal amounts of locked and unlocked funds to discourage bad actors from creating confidential assets.

## Wallet Side Shaders

Thus far, we have explored smart contracts activated by sent transactions (Chain-side Shaders). We now review the wallet-side architecture, i.e., the framework that interacts with Shaders via new transactions and retrieval of Shader metadata to Beam Wallets.

### Wallet Web Application Platform

Since v6.0, Beam Desktop Wallets have a built-in web application platform that allows first-party and third-party app development (Mobile version TBA).

### The role of Wallet Side Shaders

Due to Beam's confidential blockchain ecosystem, any blockchain-altering function requires wallets. Beam's DeFi architecture leverages the Wallet-side shader's client logic and API required for each smart contract, simplifying interactions between wallets and Chain-side Shaders

Wallet-side Shaders offer great flexibility in separating client and chain-side logic, enabling complex flows that create operations across multiple wallets through a single UI.

## Resources

### Concepts

* DLC (Discreet Log Contract)
  * [paper](https://adiabat.github.io/dlc.pdf)
  * [video](https://www.youtube.com/watch?v=P6AX8KdXAts)
  * [article](https://medium.com/@gertjaap/discreet-log-contracts-invisible-smart-contracts-on-the-bitcoin-blockchain-cc8afbdbf0db)

### Whitepapers

* [Rainbow Protocol Whitepaper](https://www.semanticscholar.org/paper/The-Rainbow-Network-%3A-An-Off-Chain-Decentralized-Robinson/03b035bc6fea62caef3455ad383cd7f8164adf2f?p2df)
* [Arwen Protocol Whitepaper](https://arwen.io/whitepaper.pdf)
* [Compound Whitepaper](https://compound.finance/documents/Compound.Whitepaper.pdf)

### Wrapped assets

* [tBTC - Bitcoin on Ethereum via Liquid](https://tbtc.network/developers/tbtc-technical-system-overview/)

### Terms

* [Collateralized Debt Position](https://defitutorials.substack.com/p/collateralized-debt-positions-cdps)
