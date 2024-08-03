# Introduction to Atomic Swaps

**For Electrum we support only Legacy type seeds and SSL connection**

**For Bitcoin use version** [**Bitcoin Core v0.17.1**](https://bitcoin.org/en/download) **.**

**For Litecoin use version** [**Litecoin v0.17.1**](https://litecoin.org/#download) **.**\



## Introduction to Atomic Swaps

Atomic Swaps offer a simple solution to the following problem: two parties (Alice and Bob) want to trade their coins without using an exchange. ‌Beam has integrated an Atomic Swap marketplace directly into your Beam wallet (desktop version only) and a simplified process to link additional cryptocurrencies from your wallet settings.

### The Non-Atomic way to Swap Cryptocurrency

The non-atomic approach to swapping cryptocurrencies consists of many arduous steps:

1. Alice wants to trade her Litecoin for Bob's Bitcoin.
2. Alice and Bob find an exchange that supports both currencies. Alice and Bob each sign up and go through the verification process.
3. Wait.
4. The verification process completes. Alice and Bob can deposit their crypto to the exchange, paying network and mining fees to do so.
5. Wait.
6. Alice and Bob's can exchange their crypto. They pay additional fees.
7. Alice now has some of Bob's bitcoin, and Bob has some of Alice's Litecoin.
8. Alice and Bob transfer their shiny new crypto coins back to their wallets, but not before paying more network and mining fees.
9. Wait some more.
10. Alice and Bob's coins have arrived in their wallets. Alice and Bob have now diversified their cryptocurrency portfolios.

This scenario is unnecessarily complex and creates lots of room for error.

With Atomic Swaps, Alice can exchange her Litecoin for Bob's ~~Bitcoin~~ _Beam coins_ directly from their respective wallets!

No verification process. No depositing to cryptocurrency exchanges. Minimal fees and wait time.

### Atomic Swaps, _Simplified_

1. Alice creates an offer for 200 of her Dogecoins for 10 of Bob's Beam coins, securing it with her secret passphrase before broadcasting the request to Bob over the Beam network.
2. Bob receives Alice's offer for 200 of her Dogecoins for 10 of his Beam coins. Bob accepts, verifying the exchange with his secret passphrase.
3. Alice receives a notification that Bob has accepted her offer.
4. Alice receives 10 Beam coins, and Bob gets 200 Dogecoin. Alice and Bob have now diversified their cryptocurrency portfolios.

### Atomic Swaps, Not-So-Simplified

1. Atomic swaps are essentially three smart contracts stacked on top of each other.
2. Each atomic swap creates three transactions between Alice and Bob: Lock transaction, Redeem transaction, and Refund transaction.
3. Both parties lock their coins on the respective chain using the Lock Transaction. In this state, the coins belong to both parties and can not be spent.
4. Once the lock is confirmed, the parties exchange secrets used to lock funds and send the Redeem transaction to the chain, effectively getting the swapped coins.
5. If the swap fails for any reason, a Refund transaction can be sent to the network after some significant period of time to get the locked coin back for each party.

