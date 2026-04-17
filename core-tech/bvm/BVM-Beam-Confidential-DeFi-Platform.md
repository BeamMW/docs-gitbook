

THIS DOCUMENT IS WORK IN PROGRESS AND MAY BE SUBJECT TO MAJOR CHANGES

# Architecture

## Introduction

Beam is a UTXO based confidential cryptocurrency based on the hybrid Mimblewimble + LelantusMW protocol. The purpose of this document is to 

## Confidential Assets

Beam Blockchain natively supports creation of new types of tokens called **Confidential Assets** (abbreviated as CA) [3]. Confidential Assets inherit all the properties of the Beam Coin (a native token on Beam Blockchain) namely, confidentiality and scalability using cut-through process. Unlike Ethereum ERC20 type contracts, Beam assets are implemented on Layer 1 and are native citizens of Beam ecosystem. Each Beam transaction can simultanieously include any number of different CA types both as inputs and as outputs, provided all inputs and outputs of the same CA type sum to zero, i.e no type conversion can occur within a transaction.

## Transaction Kernels

Each transaction in Beam consists of a set of inputs and outputs which are represented by Pedersed Commitments with adjacent zero knowledge range proofs to make sure commited values are positive and within range. In addition each transaction contains transaction Kernels, which originally contain the signed difference between the blinding factors.

Unlike inputs and outputs, which can be removed during cut-through process, Kernels forever remain in the blockchain and are essential for validation of the protocol. During Beam development, kernels were exteneded to support several different purposes, such as reflecting transaction type and supporting Lelantus shielded pool. The following extension was added to enable running Smart Contracts on Beam blockchain

## Kernels with Side Effects

A transaction kernel can specify a side effect, i.e explicit change of the node state. Such change can include transformations on the tx inputs and outputs. If the side effect modifies outputs, we say that kernel 'consumes' outputs, and the value stored in those outputs is recorded in the node state. In return, the kernel may 'emit' new inputs using the value stored in the node state. These operations effectively transform Beam into a hybrid UTXO based <> State Based cryptocurrency. 

## Beam Shaders and Beam Virtual Machine

Node state is encaspsulated witihin Beam Shaders that specify the state variables and business logic for modifying these varaibles. Beam Shaders are implement as WebAssembly[1] programs that run within Beam Virtual Machine. 

WebAssembly (abbreviated Wasm) is a binary instruction format for a stack-based virtual machine. Wasm is designed as a portable compilation target for programming languages, enabling deployment on the web for client and server applications. WebAssembly is a well known industry standard and supports wide variety of programming languages, compilers and development tools. 

## Using Beam Shaders to implement Smart Contracts

For a more technical description of Beam Smart Contracts read [here](https://github.com/BeamMW/beam/wiki/Beam-Smart-Contracts)

According to Wikipedia definition, Smart Contract [2] is a computer program or a transaction protocol which is intended to automatically execute, control or document legally relevant events and actions according to the terms of a contract or an agreement. Beam Shaders specify *implementation* of Smart Contracts on the Beam blockchain. 

Each Beam Shader is defined as a collection of state variables and methods, includng two special methods: a constructor and a destructor

Shader Constructor is only run once when the Shader is registered and provides Shader parameters which have to be supplied by the Contract Creator during the process of registering a new Shader. 

Beam Shader code and a set constructor parameters provide a **unique identification** of a Shader Instance and are recorded on the blochain. There can be only one such Shader Instance at all times. It is important to note that the same code with a different set of parameters will provide a different unique **Shader ID** and hence will constitute a different Shader Instance.

A destructor is only run once when the Beam Shader is deactivated and is reponsible for cleaning all used resources. Beam Shader can only be deactivated if it does not hold any funds so that it would not be possible to destroy value during Shader deactivation.

## Properties of Chain Side Beam Shaders

The following section describes the key properties of Beam Chain Side Shaders - i.e Shaders that are deployed on the blockchain are used for the implementation of Smart Contract logic.

### Shaders ensure bounded execution time

To avoid blocking the node by an unbounded cycle, Beam Shaders implement several limiting mechanisms including charging **execution fee** for each operation as well as enforcing a **maximum execution fee**.

### Shaders can only be invoked by transactions

The only way to execute Shader logic is by sending a transaction containing all required parameters and signatures and fees to the blockchain. Beam Shaders can not be invoked in any other automatic or scheduled manner.

### Shaders can collect fees

Most decentralized applications request specific fees collected by the application for the provided services. Shaders can collect, store and distribute such fees, including refunding excess fees to special Vault contracts.

### Shaders can call other Shaders

During their operation Shaders can delegage operations to other Shaders. All operations are performed in the context of the transaction that activated the first Shader in the chain

### Controlling Assets 

Shaders have an ability to perform emission ad burning of Confidential Assets that are controlled by the Shader. In this case assets can not be manipulated manually, all emission and burn operations are completely controlled by a transparent Shader logic. 

### Locking and Unlocking Funds

In the course of operation, Shader can either lock funds provided as transaction inputs, which are consumed in the process, or unlock funds and return them as transaction outputs that are created and added to the blockchain.


### Overdraft protection

BVM will validate that amount of locked and unlocked funds are equal so that it is not possible to create coins out of thin air, except for legal emission of the Confidential Asset which is always clearly visible on the blockchain.

## Wallet Side Shaders

So far we have described what we call Chain side Shaders - programs that are executed within the node and activated by transactions. In this section we will describe the Wallet side architecture that supports interaction and invocation of the Shaders by creating transactions as well as retrieval and presentation of the Shader data in Beam Wallets.

### Wallet Web Application Platform

Starting from v6.0, Beam Desktop (and in the future Mobile) Wallets will support built in Web Application Platform that will enable both Beam Team and third party developers to build web applications 

### The role of Wallet Side Shaders

Since Beam blockchain is completely confidential, a wallet (i.e ability to sign transactions with secret keys) is always requried to perform any state changing operation. In order to simplify and organize interactions between the wallet and the chain side shader, Beam Confidential DeFi architecture supports implementation of Wallet Side Shaders. These Shaders encapsulate client logic and the API required to interact with each specific contract and they can be implemented either by the creators of the contract or by third parties. 

As such Wallet Side Shaders provide maximum flexibility for separation of client and chain side logic. For example, they allow creating complicated Wallet Applications by creating a single UI for a group of different Chain side Shaders, including creating complex flows that perform operations across multiple different applications.





References:

[1] WebAssembly 

[2] Smart Contracts https://en.wikipedia.org/wiki/Smart_contract#:~:text=A%20smart%20contract%20is%20a,a%20contract%20or%20an%20agreement.


## Requirements

TODO

## Implementation

TODO

## Resources


### Concepts

1. DLC (Discreet Log Contract) [paper](https://adiabat.github.io/dlc.pdf), [video1](https://www.youtube.com/watch?v=P6AX8KdXAts) [article](https://medium.com/@gertjaap/discreet-log-contracts-invisible-smart-contracts-on-the-bitcoin-blockchain-cc8afbdbf0db)



### Whitepapers

1. [Rainbow Protocol Whitepaper](https://pdfs.semanticscholar.org/03b0/35bc6fea62caef3455ad383cd7f8164adf2f.pdf?_ga=2.163948086.888186934.1592119440-592008029.1592119440)

2. [Arwen Protocol Whitepaper](https://arwen.io/whitepaper.pdf)

3. [Compound Whitepaper](https://compound.finance/documents/Compound.Whitepaper.pdf)

### Wrapped assets

1. [tBTC - Bitcoin on Ethereum via Liquid](https://tbtc.network/developers/tbtc-technical-system-overview/)

### Terms

1. _Collateralized Debt Position_ ([source 1](https://defitutorials.substack.com/p/collateralized-debt-positions-cdps))
2. 