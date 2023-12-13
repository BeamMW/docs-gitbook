# Beam Documentation

## Welcome to Beam

Beam is a best-in-class confidential cryptocurrency and DeFi platform.

![Beam Confidential Cryptocurrency and BeamX Confidential DeFi Platform](/og-image.png)

Our focus is to deliver the best possible level of privacy and confidentiality without compromising on usability. This article will provide a brief overview of Beam project, what it can do, and how to get started with our platform.

Beam is a profound, engaging, and long-term project with unique and innovative technology, a great community, and ambitious goals. To learn more about Beam, you are welcome to visit our [website](/) and join us on:

* [Telegram](https://t.me/BeamPrivacy)
* [Twitter](https://twitter.com/beamprivacy)
* [Discord](http://discord.gg/BHZvAhg)
* [Reddit](https://www.reddit.com/r/beamprivacy/)
* [YouTube](https://www.youtube.com/channel/UCddqBnfSPWibf4f8OnEJm\_w)

Don't hesitate to get in touch with [Beam Support on Telegram](https://t.me/BeamSupport) or email us at support@beam.mw.

## A brief history of Beam

Beam launched in March 2018 with a singular vision: creating the best possible privacy coin that does not compromise usability. We moved quickly to assemble a team of world-class developers that shared our vision of a genuinely confidential cryptocurrency. In nine short months, our team built the first version of the Beam blockchain, and on January 3rd, 2019, we launched the Mainnet. Since then, Beam has been delivering and expanding our promise of creating a truly confidential cryptocurrency (currently on version 6.0). Our entire project is [open source](https://github.com/BeamMW) from both a coding and development process. The Beam protocol is coded entirely from scratch; not a hard fork or clone of another project.

Beam is a deflationary coin with capped emission and periodic halving every four years (similar to bitcoin). Beam uses a Proof-of-Work consensus algorithm called BeamHash III (created by Wilke Trei, a.k.a., Lolliedieb, [pdf](https://docs.beam.mw/Beam\_Hash\_III\_Slides.pdf)) with emission starting from zero (e.g., no pre-mine or ICO) and is currently mined on GPUs. See our [mining guide](/docs/mining) for more information on how-to mine Beam coins.

During the last three years of the project's existence, we have made many improvements and additions to our technological stack. Currently, Beam uses a combination of Mimblewimble and Lelantus protocol on layer 1, with transactions routed through Dandelion++ and transactions merging in stem phase on Layer 0 for ultimate privacy. To learn more about Beam technology, please refer to our [wiki ](https://github.com/BeamMW/beam/wiki)page.

{% embed url="https://www.youtube.com/watch?v=oNsT3M6NWTI" %}

### Beam Wallets

Beam implemented sleek, beautiful, and user-friendly wallets for all platforms ([Windows](/downloads/), [Mac](/downloads/), [Linux](/downloads/), [iOS](https://itunes.apple.com/us/app/beam-privacy-wallet/id1459842353?ls=1\&mt=8), and [Android](https://play.google.com/store/apps/details?id=com.mw.beam.beamwallet.mainnet)). To get started using Beam wallets, proceed to [downloads](http://beam.mw/downloads/) and refer to our [Desktop ](/docs/desktop)or [Mobile ](/docs/mobile)User Guides.

![](<.gitbook/assets/Main screen top.PNG>)

Let's revisit the major functionalities supported by Beam nodes and Beam wallets:

### Private Transactions

All Beam transactions are **private by default**, which means no one can see the Sender or Receiver of the transaction or the transaction's value (except, of course, for the participants of the transaction itself). **No addresses or other identity-related information is ever recorded on the blockchain**, which means the user's transaction history always remains private. This level of privacy places Beam in a class of its own.

### Atomic Swaps

Beam has built an Atomic Swap functionality that allows peer-to-peer trading of Beam for Bitcoin, Ethereum, Litecoin, DOGE (and more) in a completely decentralized way without an exchange or any other 3rd party intermediary. Atomic Swaps are the pinnacle of decentralization, censorship resistance, and core values of cryptocurrencies. In our usual way, we have made them easy to use by integrating an Atomic Swap marketplace into our wallets and supporting a simplified process to set up the Atomic Swaps from wallet settings.

### Direct Payment Channels (a.k.a. Laser Beam)

Beam supports creating direct payment channels between wallets that work similarly to Lightning Network payment channels. The purpose of those channels is to allow instant payments along the funded channel between two wallets with almost instant confirmation time and later on-chain settlement. We called this feature "Laser Beam," and it is perfect for streaming and micropayments.

### Tokens (a.k.a. Beam Confidential Assets) <a href="#tokens-a-k-a-beam-confidential-assets" id="tokens-a-k-a-beam-confidential-assets"></a>

The Beam blockchain supports creating Confidential Assets, which just like Beam are **private by default**. This differs from Ethereum, which supports creating new tokens using Smart Contracts (e.g., ERC-20 and others), Confidential Assets on Beam boasts a **Layer 1** **implementation as a first-class citizen** with native support offered by all nodes, wallets, and Dapps. This implementation makes new asset creation more effortless, as the security and reliability would not require verifying and auditing a different smart contract for each new asset. To create a new asset on Beam, please refer to [this guide](https://www.raskul.com/creating-a-confidential-asset-on-beam/).

### Decentralized Apps (a.k.a. Dapps) and Dapp Store

Dapps are implemented via Smart Contracts (read below), they can be embedded into a Beam wallet or run on a standalone website. Expect multiple financial instruments and games to arrive in the next months!

Beam Wallets have a built-in Decentralized Application (DAPP) store that provides easy access for users and supports APIs that allow seamless integration and discovery of new apps.

### **Smart Contracts (a.k.a. Beam Shaders)** <a href="#smart-contracts-a-k-a-beam-shaders" id="smart-contracts-a-k-a-beam-shaders"></a>

Beam Shaders enable distributed DeFi applications to run through the Beam Virtual Machine (BVM) built into all Beam nodes and wallets. The Dapp logic implements through Smart Contracts that are programmable with any language compilable to WASM (read: practically every language). The developer toolchain currently works for C++ with additional programming languages (Go, Rust, etc.) planned for future releases.

Beam Shaders is a relatively new feature, and the documentation is still in progress. To learn more about creating DeFi applications on Beam, please refer to [these guides](https://github.com/BeamMW/beam/wiki#beam-confidential-defi-platform-in-progress) and join the Beam [Developers community on Telegram](https://t.me/beamdevsupport).

### Beam Ethereum Bridges

With great fanfare, we can announce that the Beam Bridge to Ethereum is now available on [**Mainnet**](/downloads/)! Users can seamlessly transfer assets across Beam and Ethereum blockchains using decentralized privacy-preserving cross-chain transactions. For more information on how to utilize Beam Bridges for cross-chain transactions, refer to our [**Bridges guide**](/docs/bridges/)!

## Beam Essentials

If you are new to Beam, there are some essential concepts that you need to know and understand before you get started. Since Beam is an innovative confidential cryptocurrency, some things work differently from other coins you might have used. Knowing those differences is crucial to avoid problems and have the best possible Beam experience.

### Beam is a new and experimental technology

Cryptocurrency is new and very experimental technology subject to volatile fluctuations in value, and Beam is no exception. Despite our best efforts to debug, test, and audit our software, bugs do happen. This technology should be used carefully and at your own risk. The flip side of the decentralized nature of crypto is that you and only you are responsible for the safety of your funds.

### Keep your seed phrase safe at all times

The seed phrase is a twelve-word mnemonic sequence used to generate all of your secret keys. Your seed phrase generates when you create a new wallet for the first time. Record and keep your seed phrase safe at all times. Losing or exposing your seed phrase might result in losing access to all funds. If you have any suspicion that your seed phrase is compromised, please follow the [Exposed seed or wallet](./#exposed-seed-or-wallet) guide.

### Always use strong passwords for your wallets

Beam stores the wallet-related data in a local database file encrypted with your password. If someone else gains access to your computer or mobile phone, either physically or remotely, the database file can be accessed and hacked. We strongly recommend a password minimum of 16 characters/symbols for wallets expected to hold significant amounts of funds either now or in the future. If you have any suspicion that your wallet is compromised, please follow the [Exposed seed or wallet](./#exposed-seed-or-wallet) guide.

### Beam Transaction types

Even though all Beam transactions are confidential by default, there are three different transaction types:

**Online transactions** (recommended in most cases) have the lowest fees of any transaction type. Those transactions are created interactively by both wallets and require that both Sender and Receiver wallets are online within 12 hours of transaction creation. In most cases, both wallets are online when sending or receiving payments; the 12 hour time window offers users convenience and flexibility. The typical flow of such transaction usually looks as follows:

1. The Receiver launches their wallet, clicks "Receive," then copies the created "Regular" address before sending it to the Sender via any communication channel (email, Telegram, or other messenger apps). The Receiver leaves their wallet open as they wait for Sender.
2. The Sender launches their wallet, clicks "send." In the "Send to" field, pastes the received address. The Sender enters the amount of Beam coins to send and clicks "send."
3. The wallets then communicate with each other behind the scenes to complete the transaction. The Receiver's wallet automatically accepts the transaction, signs it, and sends it back to the Sender.
4. The Sender wallet completes the transaction and sends it to the network. The transaction will reflect the Sender/Reciever wallet balance after the block containing the transaction is added to the blockchain. Your transaction will read from "in progress" to "completed." 

**Offline transactions** have higher fees than online transactions. Suppose the Sender knows that the Receiver will not be online within the next 12 hours and has no way (or will) to communicate with the Sender. In this case, the Sender can use a previously received address to send funds without any additional communication with the Receiver. The Receiver will see those funds the next time they open their wallets.

**Max Privacy transactions** is a special Offline transaction that uses the **maximum anonymity set**. In other words, it guarantees the user maximum privacy. Max Privacy transactions can take up to 72 hours to complete. Users can adjust the [lock times](/docs/mobile/settings#privacy) in their [wallets](/docs/desktop/settings#privacy). However, shorter lock times equals less privacy; the highest lock time guarantees your private information remains confidential.

### Beam Addresses

Even though Beam uses the term "Address" for convenience, Beam addresses are different from most other cryptocurrencies. Beam Addresses are not recorded on the blockchain and are only used to communicate with other wallets to create different transaction types. Beam wallets will generate an address for every new transaction (recommended for most cases) and support three types of addresses:

**Regular addresses** will function for Online and Offline transactions. The Receiver generates and shares a Regular address with the Sender. Then the Sender decides to make the payment Offline or Online before completing the transaction. Offline payments offer users flexibility if both parties cannot be online within 12 hours. In some (rare) cases, the Regular address will expire, and both parties will need to restart the transaction.

**Max anonymity set** is a special Offline address that guarantees users maximum privacy. If the Receiver shares a Max anonymity address with Sender, the transaction type will change to [**Max Privacy**](./#beam-transaction-types).

**Public Offline address** is a unique address connected to your Beam wallet that never expires. Public Offline addresses are a little less secure in one aspect: the Sender can potentially see when the Receiver spends the funds sent to this address (though, of course, not to whom or how much). Since Beam is all about privacy, we only recommend using this address for donations or a Receiver wallet that is rarely online.

### Transaction history, Address book, and metadata are stored locally in each wallet

Unlike most cryptocurrencies, Beam does not record your transaction history, wallet addresses, and other transaction-related data on the blockchain. That data is stored locally on your device, so data in your wallet does not reflect as data stored on the blockchain. If you are deleting your Beam wallet but wish to preserve the metadata, you should back up or export your wallet database before deleting your wallet.

Your wallet balance and transaction history can have inconsistencies if you have multiple wallets on different devices using one seed phrase.

{% embed url="https://www.youtube.com/watch?v=dm21bOh1TVg" %}

## Getting Started

* [Download](https://www.beam.mw/downloads) Beam wallet (available on every platform).
* Read the guide for [desktop](/docs/desktop/) and [mobile](/docs/mobile) wallets.
* Get live [support](https://t.me/BeamSupport) if you run into any issues or have questions.

## FAQ

### Exposed seed or wallet

If you have even the slightest suspicion that either your seed or wallet database file could have been exposed, follow these next steps immediately:

1. Start the potentially exposed wallet or restore the wallet using the seed phrase on a clean device and verify your wallet balance hasn't changed
2. Create a new wallet and record the new seed phrase.
3. Transfer your coins from the old wallet to the new one and wait for the transaction to complete.
4. Once your transaction reads from "in progress" to "completed," you can be confident that your funds are safe, but we are still not done.
5. Inform anyone you expect to receive payments from (Offline, Max Privacy, Public Offline), and provide them with new addresses from your new Beam wallet. Failure to do so might result in payments sent to your compromised wallet addresses.

* How to restore Beam wallet from seed phrase ([desktop](/docs/desktop/restore-wallet), [mobile](/docs/mobile/how-to-restore-wallet)).
* [How Addresses work in Beam.](./#beam-addresses)
* [What are Max Privacy transactions?](./#beam-transaction-types)

* [Glossary of Beam terms](/docs/beampedia)
