# Beam Confidential DeFi Platform

[BVM Internals](https://github.com/BeamMW/beam/wiki/bvm/BVM-Internals)

[BVM Host Functions Reference](https://github.com/BeamMW/beam/wiki/bvm/BVM-functions-for-shaders)

[Web wallet client](https://github.com/BeamMW/beam/wiki/WASM-wallet-client)

# Documentation

[User Guides](https://beam.mw/docs)

[Exchange Integration Guide](https://github.com/BeamMW/beam/wiki/Exchange-Pool-integration-guide)

[How To Build](https://github.com/BeamMW/beam/wiki/How-to-build)

---

## Core — Cryptography, Transactions, Block Structure

Reference documentation for Beam's cryptographic primitives, transaction anatomy, block format, and chain state structures.

* [Beam Technical Specifications](https://github.com/BeamMW/beam/wiki/Beam-Technical-Specifications)
* [Cryptographic Primitives](https://github.com/BeamMW/beam/wiki/core/Core-Cryptographic-Primitives)
* [Core Transaction Elements](https://github.com/BeamMW/beam/wiki/core/Core-transaction-elements)
* [Merkle Structures](https://github.com/BeamMW/beam/wiki/core/Core-Merkle-Structures)
* [Block and Chain State](https://github.com/BeamMW/beam/wiki/core/Core-Block-And-Chain-State)

## Node — Architecture, P2P, Mining, Sync

Full-node internals: block processing, transaction pool, peer-to-peer protocol, proof-of-work, and synchronization.

* [Node Architecture](https://github.com/BeamMW/beam/wiki/node/Node-Architecture)
* [P2P Network Protocol](https://github.com/BeamMW/beam/wiki/node/Node-P2P-Protocol)
* [Fly Client Protocol (SPV)](https://github.com/BeamMW/beam/wiki/node/Node-Fly-Client-Protocol)
* [Beam Mining](https://github.com/BeamMW/beam/wiki/BEAM-Mining)
  * [Mining Modes](https://github.com/BeamMW/beam/wiki/node/Node-Mining-Modes)
  * [AVX Optimization](https://github.com/BeamMW/beam/wiki/AVX)
  * [Supported nVidia Cards (OpenCL)](https://github.com/BeamMW/beam/wiki/Supported-nVidia-cards-for-mining-using-OpenCL-miner)

## Wallet — Engine, DB, Addresses, Key Management

Wallet architecture, database schema, key derivation, address formats, SBBS messaging, and hardware wallet support.

* [Wallet Architecture](https://github.com/BeamMW/beam/wiki/wallet/Wallet-Architecture)
* [Wallet Database Schema](https://github.com/BeamMW/beam/wiki/wallet/Wallet-Database-Schema)
  * [Payment Confirmation](https://github.com/BeamMW/beam/wiki/Payment-confirmation-(proof))
  * [One-Side Payment](https://github.com/BeamMW/beam/wiki/One-side-payments)
  * [Transactions over TOR](https://github.com/BeamMW/beam/wiki/transactions/Transactions-with-Beam-Wallet-CLI-over-TOR-network)
* [Addresses and Key Derivation](https://github.com/BeamMW/beam/wiki/wallet/Wallet-Addresses-And-Key-Derivation)
* [Secure Bulletin Board System (SBBS)](https://github.com/BeamMW/beam/wiki/wallet/Wallet-SBBS)
* [WASM Wallet Client](https://github.com/BeamMW/beam/wiki/WASM-wallet-client)
* [Wallet Audit (Read-Only)](https://github.com/BeamMW/beam/wiki/wallet/Wallet-audit)
  * [Setting Up Read-Only Wallet](https://github.com/BeamMW/beam/wiki/Setting-up-read-only-wallet-for-monitoring)
* [Key Keeper and Hardware Wallet Support](https://github.com/BeamMW/beam/wiki/wallet/Wallet-Key-Keeper)
  * [Hardware Wallet Design](https://github.com/BeamMW/beam/wiki/HW-wallet-design)
  * [How to Test with Trezor T](https://github.com/BeamMW/beam/wiki/How-to-test-Beam-with-Trezor-wallet)
* [Beam URI Scheme](https://github.com/BeamMW/beam/wiki/Beam-URI-scheme)

## Transactions — Simple, Assets, Shielded, Swaps, Channels

Transaction types and creation protocols: plain BEAM transfers, Confidential Assets, Lelantus shielded pool, atomic swaps, hi-frequency transactions, and payment channels.

* [Transaction Creation Protocol](https://github.com/BeamMW/beam/wiki/transactions/Transactions-Creation-Protocol)
  * [Token Format](https://github.com/BeamMW/beam/wiki/Atomic-swap-token)
* [Simple Transactions and Confidential Assets](https://github.com/BeamMW/beam/wiki/transactions/Transactions-Confidential-Assets)
  * [Asset Descriptor v1.0](https://github.com/BeamMW/beam/wiki/Asset-Descriptor-v1.0)
* [Lelantus-MW Shielded Pool](https://github.com/BeamMW/beam/wiki/transactions/Transactions-Lelantus-Shielded-Pool)
  * [Confidential Lelantus Assets (MW-CLA)](https://github.com/BeamMW/beam/wiki/MW-CLA)
* [Hi-Frequency Transactions (HFTX)](https://github.com/BeamMW/beam/wiki/transactions/Transactions-Hi-Frequency)
* [Atomic Swaps](https://github.com/BeamMW/beam/wiki/transactions/Transactions-Atomic-Swaps)
* [Asset Swaps / DEX](https://github.com/BeamMW/beam/wiki/transactions/Transactions-Assets-Swaps)
* [Laser Channels (Payment Channels)](https://github.com/BeamMW/beam/wiki/transactions/Transactions-Laser-Channels)
  * [Laser Beam Commands](https://github.com/BeamMW/beam/wiki/Laser-BEAM-commands)
* [Transaction Ordering and Front-Running Protection](https://github.com/BeamMW/beam/wiki/historical/Transaction-ordering-and-front-running-protection)

## BVM — Smart Contracts, Shaders, IPFS

Beam Virtual Machine internals, shader (smart contract) development SDK, IPFS integration, and EVM compatibility.

* [BVM Internals](https://github.com/BeamMW/beam/wiki/bvm/BVM-Internals)
* [Smart Contracts Overview](https://github.com/BeamMW/beam/wiki/bvm/BVM-Beam-Smart-Contracts)
* [BVM Host Functions Reference](https://github.com/BeamMW/beam/wiki/bvm/BVM-functions-for-shaders)
* [Building Beam Shaders](https://github.com/BeamMW/beam/wiki/bvm/BVM-Building-Beam-Shaders)
* [Running Shaders with CLI Wallet](https://github.com/BeamMW/beam/wiki/bvm/BVM-Running-Beam-Shaders-using-CLI-Wallet)
* [Shader SDK Index](https://github.com/BeamMW/beam/wiki/bvm/BVM-Shader-SDK-Index)
* [Ethash Verification in Contracts](https://github.com/BeamMW/beam/wiki/Ethash-verification-in-contracts)
* [BEAM IPFS Support](https://github.com/BeamMW/beam/wiki/BEAM-IPFS-Support)

**BeamX Contract Examples**
* [BeamX Getting Started](https://github.com/BeamMW/beam/wiki/BeamX-Getting-Started)
* [Using BeamX Faucet with CLI Wallet](https://github.com/BeamMW/beam/wiki/Using-BeamX-Faucet-contract-with-CLI-Wallet)
* [Using BeamX Vault with CLI Wallet](https://github.com/BeamMW/beam/wiki/Using-BeamX-Vault-contract-with-CLI-Wallet)
* [Using BeamX Roulette with CLI Wallet](https://github.com/BeamMW/beam/wiki/Using-BeamX-Roulette-contract-with-CLI-Wallet)

## Consensus — Hard Forks, BeamHash, Upgrade Guides

Consensus parameter evolution, proof-of-work algorithm history, and network upgrade guides for pools and exchanges.

* [Hard Forks — Rules, Heights, and Consensus Changes](https://github.com/BeamMW/beam/wiki/consensus/Consensus-Hard-Forks)
* [BeamHash PoW Algorithm](https://github.com/BeamMW/beam/wiki/consensus/Consensus-BeamHash)
* [Beam Warp: dPoS / PBFT Consensus](https://github.com/BeamMW/beam/wiki/consensus/Consensus-Beam-Warp-dPoS)
* [Upgrade Guide: Eager Electron 5.0](https://github.com/BeamMW/beam/wiki/Beam-Eager-Electron-5.0-Upgrade-Guide-for-pools-and-exchanges)
* [Upgrade Guide: Fierce Fermion 6.0](https://github.com/BeamMW/beam/wiki/Beam-Fierce-Fermion-6.0-Upgrade-Guide-for-pools-and-exchanges)
* [Testing Hard Forks on Local Testnet](https://github.com/BeamMW/beam/wiki/Testing-Beam-Hard-Fork-on-Local-Testnet)

## API — Wallet, Explorer, Stratum

JSON-RPC and protocol API references for wallet integration, blockchain data access, and mining pool operation.

* [Beam Wallet API](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API)
  * [v6.0](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API-v6.0)
  * [v6.1](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API-v6.1)
  * [v6.2](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API-v6.2)
  * [v7.0](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API-v7.0)
  * [v7.1](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API-v7.1)
  * [v7.2](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API-v7.2)
  * [v7.3](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API-v7.3)
  * [v7.4](https://github.com/BeamMW/beam/wiki/api/Beam-wallet-protocol-API-v7.4)
* [Beam Node Explorer API](https://github.com/BeamMW/beam/wiki/api/Beam-Node-Explorer-API)
* [Beam Mining API (Stratum)](https://github.com/BeamMW/beam/wiki/api/Beam-mining-protocol-API-(Stratum))

## Contributing — C++ Conventions and Codebase Guide

Conventions, idioms, and patterns used throughout the Beam C++ codebase.

* [C++ Style and Conventions](https://github.com/BeamMW/beam/wiki/Beam-Cpp-Style-And-Conventions)
* [How To Build](https://github.com/BeamMW/beam/wiki/How-to-build)
* [Contribution Guidelines](https://github.com/BeamMW/beam/wiki/Contribution-Guidelines)

---

# Research and Historical Proposals

Design proposals and research documents preserved for historical reference. These were not fully implemented in their described form.

* [Eliminating Transaction Kernels](https://github.com/BeamMW/beam/wiki/historical/Thoughts-about-eliminating-transaction-kernels)
* [Wallets Discovery and Dialog Proposal](https://github.com/BeamMW/beam/wiki/historical/Wallets-discovery-and-dialog-proposal)
* [Proposal for I/O Layer and P2P](https://github.com/BeamMW/beam/wiki/historical/Proposal-for-I-O-layer-and-P2P)
* [Mimblewimble Whitepaper (June 2016)](https://github.com/BeamMW/beam/wiki/Mimblewimble-Whitepaper-(June-2016))
* [Beam Position Paper](https://github.com/BeamMW/beam/wiki/historical/Beam-Position-Paper)
* [News Channels](https://github.com/BeamMW/beam/wiki/Beam-news-channels)
