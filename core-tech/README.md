# Beam Confidential DeFi Platform

[BVM Internals](bvm/BVM-Internals.md)

[BVM Host Functions Reference](bvm/BVM-functions-for-shaders.md)

[Web wallet client](WASM-wallet-client.md)

# Documentation

[User Guides](https://beam.mw/docs)

[Exchange Integration Guide](Exchange-Pool-integration-guide.md)

[How To Build](How-to-build.md)

---

## Core — Cryptography, Transactions, Block Structure

Reference documentation for Beam's cryptographic primitives, transaction anatomy, block format, and chain state structures.

* [Beam Technical Specifications](Beam-Technical-Specifications.md)
* [Cryptographic Primitives](core/Core-Cryptographic-Primitives.md)
* [Core Transaction Elements](core/Core-transaction-elements.md)
* [Merkle Structures](core/Core-Merkle-Structures.md)
* [Block and Chain State](core/Core-Block-And-Chain-State.md)

## Node — Architecture, P2P, Mining, Sync

Full-node internals: block processing, transaction pool, peer-to-peer protocol, proof-of-work, and synchronization.

* [Node Architecture](node/Node-Architecture.md)
* [P2P Network Protocol](node/Node-P2P-Protocol.md)
* [Fly Client Protocol (SPV)](node/Node-Fly-Client-Protocol.md)
* [Beam Mining](BEAM-Mining.md)
  * [Mining Modes](node/Node-Mining-Modes.md)
  * [Supported nVidia Cards (OpenCL)](historical/Supported-nVidia-cards-for-mining-using-OpenCL-miner.md)

## Wallet — Engine, DB, Addresses, Key Management

Wallet architecture, database schema, key derivation, address formats, SBBS messaging, and hardware wallet support.

* [Wallet Architecture](wallet/Wallet-Architecture.md)
* [Wallet Database Schema](wallet/Wallet-Database-Schema.md)
  * [Payment Confirmation](historical/Payment-confirmation-(proof).md)
  * [One-Side Payment](historical/One-side-payments.md)
  * [Transactions over TOR](transactions/Transactions-with-Beam-Wallet-CLI-over-TOR-network.md)
* [Addresses and Key Derivation](wallet/Wallet-Addresses-And-Key-Derivation.md)
* [Secure Bulletin Board System (SBBS)](wallet/Wallet-SBBS.md)
* [WASM Wallet Client](WASM-wallet-client.md)
* [Wallet Audit (Read-Only)](wallet/Wallet-audit.md)
  * [Setting Up Read-Only Wallet](historical/Setting-up-read-only-wallet-for-monitoring.md)
* [Key Keeper and Hardware Wallet Support](wallet/Wallet-Key-Keeper.md)
  * [Hardware Wallet Design](HW-wallet-design.md)
  * [How to Test with Trezor T](How-to-test-Beam-with-Trezor-wallet.md)
* [Beam URI Scheme](Beam-URI-scheme.md)

## Transactions — Simple, Assets, Shielded, Swaps, Channels

Transaction types and creation protocols: plain BEAM transfers, Confidential Assets, Lelantus shielded pool, atomic swaps, hi-frequency transactions, and payment channels.

* [Transaction Creation Protocol](transactions/Transactions-Creation-Protocol.md)
* [Simple Transactions and Confidential Assets](transactions/Transactions-Confidential-Assets.md)
  * [Asset Descriptor v1.0](Asset-Descriptor-v1.0.md)
* [Lelantus-MW Shielded Pool](transactions/Transactions-Lelantus-Shielded-Pool.md)
  * [Confidential Lelantus Assets (MW-CLA)](MW-CLA.md)
* [Hi-Frequency Transactions (HFTX)](transactions/Transactions-Hi-Frequency.md)
* [Atomic Swaps](transactions/Transactions-Atomic-Swaps.md)
  * [Swap Token Format](transactions/Transactions-Atomic-Swaps.md#swap-token-format)
* [Asset Swaps / DEX](transactions/Transactions-Assets-Swaps.md)
* [Laser Channels (Payment Channels)](transactions/Transactions-Laser-Channels.md)
  * [CLI Reference](transactions/Transactions-Laser-Channels.md#cli-reference)
* [Transaction Ordering and Front-Running Protection](historical/Transaction-ordering-and-front-running-protection.md)

## BVM — Smart Contracts, Shaders, IPFS

Beam Virtual Machine internals, shader (smart contract) development SDK, IPFS integration, and EVM compatibility.

* [BVM Internals](bvm/BVM-Internals.md)
* [Smart Contracts Overview](bvm/BVM-Beam-Smart-Contracts.md)
* [BVM Host Functions Reference](bvm/BVM-functions-for-shaders.md)
* [Building Beam Shaders](bvm/BVM-Building-Beam-Shaders.md)
* [Running Shaders with CLI Wallet](bvm/BVM-Running-Beam-Shaders-using-CLI-Wallet.md)
* [Shader SDK Index](bvm/README.md)
* [Ethash Verification in Contracts](historical/Ethash-verification-in-contracts.md)
* [BEAM IPFS Support](BEAM-IPFS-Support.md)

**BeamX Contract Examples**
* [BeamX Getting Started](historical/BeamX-Getting-Started.md)
* [Using BeamX Faucet with CLI Wallet](historical/Using-BeamX-Faucet-contract-with-CLI-Wallet.md)
* [Using BeamX Vault with CLI Wallet](historical/Using-BeamX-Vault-contract-with-CLI-Wallet.md)
* [Using BeamX Roulette with CLI Wallet](historical/Using-BeamX-Roulette-contract-with-CLI-Wallet.md)

## Consensus — Hard Forks, BeamHash, Upgrade Guides

Consensus parameter evolution, proof-of-work algorithm history, and network upgrade guides for pools and exchanges.

* [Hard Forks — Rules, Heights, and Consensus Changes](consensus/Consensus-Hard-Forks.md)
* [BeamHash PoW Algorithm](consensus/Consensus-BeamHash.md)
* [Beam Warp: dPoS / PBFT Consensus](consensus/Consensus-Beam-Warp-dPoS.md)
* [Upgrade Guide: Eager Electron 5.0](historical/Beam-Eager-Electron-5.0-Upgrade-Guide-for-pools-and-exchanges.md)
* [Upgrade Guide: Fierce Fermion 6.0](historical/Beam-Fierce-Fermion-6.0-Upgrade-Guide-for-pools-and-exchanges.md)

## API — Wallet, Explorer, Stratum

JSON-RPC and protocol API references for wallet integration, blockchain data access, and mining pool operation.

* [Beam Wallet API](api/README.md)
  * [v6.0](api/Beam-wallet-protocol-API-v6.0.md)
  * [v6.1](api/Beam-wallet-protocol-API-v6.1.md)
  * [v6.2](api/Beam-wallet-protocol-API-v6.2.md)
  * [v7.0](api/Beam-wallet-protocol-API-v7.0.md)
  * [v7.1](api/Beam-wallet-protocol-API-v7.1.md)
  * [v7.2](api/Beam-wallet-protocol-API-v7.2.md)
  * [v7.3](api/Beam-wallet-protocol-API-v7.3.md)
  * [v7.4](api/Beam-wallet-protocol-API-v7.4.md)
* [Beam Node Explorer API](api/Beam-Node-Explorer-API.md)
* [Beam Mining API (Stratum)](api/Beam-mining-protocol-API-(Stratum).md)

## Contributing — C++ Conventions and Codebase Guide

Conventions, idioms, and patterns used throughout the Beam C++ codebase.

* [C++ Style and Conventions](Beam-Cpp-Style-And-Conventions.md)
* [How To Build](How-to-build.md)
* [Contribution Guidelines](Contribution-Guidelines.md)

---

# Research and Historical Proposals

Design proposals and research documents preserved for historical reference. These were not fully implemented in their described form.

* [Eliminating Transaction Kernels](historical/Thoughts-about-eliminating-transaction-kernels.md)
* [Wallets Discovery and Dialog Proposal](historical/Wallets-discovery-and-dialog-proposal.md)
* [Proposal for I/O Layer and P2P](historical/Proposal-for-I-O-layer-and-P2P.md)
* [Mimblewimble Whitepaper (June 2016)](Mimblewimble-Whitepaper-(June-2016).md)
* [Beam Position Paper](historical/Beam-Position-Paper.md)
* [News Channels](Beam-news-channels.md)
