# Getting started

## Welcome to Beam Developer Portal

<mark style="color:red;">**NOTE: Beam Developer Portal is currently under construction and is subject to changes**</mark>

Beam is a confidential, scalable cryptocurrency and DeFi platform designed entirely in C++. Since its inception in 2018, Beam has undergone significant development and growth, becoming a large and intricate project.

This developer portal will provide comprehensive information about the development of both Beam and Smart Contracts (also known as Shaders) on the Beam blockchain but also includes tutorials to assist developers in creating these features.

**Below are some key resources that you may find essential in your journey:**

* [Beam Source Code](https://github.com/BeamMW)
* [Developer Support Channel on Telegram](https://t.me/beamdevsupport): for any questions you might have along the way.
* [Building Beam](guides/building-beam.md): covers the process of building all Beam components from the source files.

**Topics covered will include:**

* [Core Technology Overview](./#core-technology-overview)
* [Confidential Assets](./#confidential-assets)
* [Smart Contracts (aka Shaders)](./#smart-contracts)
* [Wallet API (and integration guides)](./#wallet-api)
* [Web Wallet ](./#web-wallet)
* [Desktop Wallet](./#desktop-wallet)
* [Node](./#node)

### Core Technology Overview

In the beginning, Beam was originally designed as an implementation of the [Mimblewimble ](beam-technology/mimblewimble.md)([original whitepaper](https://docs.beam.mw/Mimblewimble.pdf)) protocol. Since then, Beam version 5.0 included support for a [Lelantus](https://github.com/BeamMW/beam/wiki/Lelantus-MW) shielded pool which became the [LelantusMW](beam-technology/lelantusmw.md) protocol that is the core of Beam's Privacy Layer one.

The Beam network uses the [Dandelion protocol](https://blockonomi.com/dandelion-protocol/), merging non-interactive transactions along the stem phase. Additional technical information regarding this implementation can be found [here](https://github.com/BeamMW/beam/wiki/Transaction-graph-obfuscation).

**Beam also features:**

* [Atomic Swaps](beam-technology/atomic-swaps.md): Allows the exchange of Beam for Bitcoin, Ethereum, and several other currencies without needing a trusted third party.
* Laser Beam: A direct payment channel implementation similar to those used in the [Lightning Network](https://lightning.network/).
* Secure Bulletin Board System ([SBBS](beam-technology/sbbs.md)): An encrypted, decentralized messaging system that broadcasts information and creates transactions between Beam wallets.

### Confidential Assets

Beam offers the creation of new asset types on the blockchain called Confidential Assets (CA). Each new CA is designated as a first-class citizen in Beam nodes due to the Layer 1 implementation and does not require using Smart Contracts like Ethereum ERC-20 tokens.

**This results in Beam tokens offering the same privacy and security as Beam coins** while seamlessly integrating with Beam Wallets and dApps. Confidential Assets can be emitted manually or deployed via Smart Contracts within Beam Virtual Machine.

For more information, refer to this guide's dedicated [Confidential Assets](confidential-assets/) section.

### Beam Shaders

Beam version 6.0 included a tweaked BVM to offer Smart Contract support, aka Beam Shaders. Beam Shaders offers a flexible Smart Contract development due to the BVM’s native operating of WebAssembly compatible langues, i.e., C++, Rust, Go, etc.

Beam Shaders, BVM, and all related tools are the pillars of [Beam’s Confidential DeFi Platform](using-beam-shaders.md).

### Wallet API

Beam’s robust Wallet API facilitates integration with exchanges, payment services, and third-party applications. While other networks offer APIs directly from the node, Beam API is coded directly into the Beam wallet. Using a wallet and secret keys ensures sensitive information remains secure without compromising integration with the Beam network.

For more information regarding Wallet API, refer to this [Beam wiki](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API).

### Web Wallet

Beam offers a simplified version of the Beam Desktop Wallet in a web wallet metamask that runs in the background of [Google Chrome](https://chrome.google.com/webstore/search/beam). The Beam Web Wallet allows users to store, send and receive Beam coins but does not currently offer Atomic Swaps or dApp support.

All wallet balances will be reflected accurately across all devices connected with your seed phrase (Desktop wallet, Mobile wallet, Web Wallet, etc.). However, web wallet metadata is not available for export.

Refer to this [web wallet guide](http://localhost:5000/o/-M\_G-3uaAbZMXViHLVZl/s/PPXXGuwTXZfforCZnW03/) for more information on getting started.

### Desktop Wallet

The Beam Desktop Wallet offers advanced privacy and security features in a user-friendly interface that allows users to hold, send or receive Beam, BeamX coins, and other confidential assets. The desktop wallet ([currently version 7.3](/downloads/mainnet-mac)) is available on Mac, Windows, and Linux platforms and provides access to Beam’s advanced features like dApps, and Atomic swaps.

Refer to our [desktop wallet guide](http://localhost:5000/o/-M\_G-3uaAbZMXViHLVZl/s/-MaSm2QyXLRMULOUF0cE/) for more information!

### Node

Beam Node is an essential component of the Beam blockchain. Each individual node creates a peer-to-peer network that validates transactions and blocks on the blockchain. Whether mining or validating nodes, each serves a crucial role in maintaining the integrity of the Beam Network.

Refer to [this guide](http://localhost:5000/o/-M\_G-3uaAbZMXViHLVZl/s/-MahNa0IaGxTr8GkB5rz/) for more information on creating your own node within the Beam network.
