---
description: >-
  The command line guide for Atomic Swaps, i.e., exchanging two different
  cryptocurrencies from separate blockchains in a trust-less and decentralized
  manner.
---

# Atomic Swaps CLI User Guide

Atomic Swaps are a peer-to-peer marketplace built directly into your Beam wallet that allows users to swap Beam for another cryptocurrency. Trade your Beam for Bitcoin, Ehtereum, LiteCoin, DOGE, and more!

{% hint style="danger" %}
**Atomic Swaps is in beta.**

**DO NOT USE THIS HIGHLY ADVANCED FEATURE FOR LARGE SUMS OF MONEY**

**For Electrum we support only Legacy type seeds and SSL connection**

**For Bitcoin use version**[**`Bitcoin_core_v0.17.1`**](https://bitcoin.org/en/download)**.**

**For Litecoin use version**[**`Litecoin_v0.17.1`**](https://litecoin.org/#download)**.**
{% endhint %}

## Perform an Atomic Swap with LiteCoin

Using Alice and Bob as an example, we will go over how to perform an Atomic Swap of Alice's Beam for Bob's LiteCoin (LTC).

### Run a full LTC node

Bob and Alice must configure their LTC node to allow RPC access either through the command line or edit their`config`file according to the [LTC documentation](https://litecoin.info/index.php/Litecoin.conf) (for this example, we will use standard node and RPC ports).

Alice and Bob can run a full LTC node by entering the following command:

```
 ./litecoind -server -datadir="path_to_litecoin_wallet_data" -rpcuser= -rpcpassword= -printtoconsole
```

#### Alice's LTC node

```
./litecoind -server -datadir="Alice/path_to_litecoin_wallet_data" -rpcuser=Alice -rpcpassword=123 -printtoconsole
```

#### Bob's LTC node

```
 ./litecoind -server -datadir="Bob/path_to_litecoin_wallet_data" -rpcuser=Bob -rpcpassword=123 -printtoconsole
```

{% hint style="danger" %}
#### The nodes must be synced to the current blockchain height before an Atomic Swap can occur.
{% endhint %}

### Run a full Beam node

Please refer to this [guide](https://beamx.gitbook.io/cli-guide/#synching-with-a-specific-node) for setting up a full Beam node.

Users can check their LTC balance by entering the following command:

```
./litecoin-cli -rpcuser= -rpcpassword= getbalance
```

#### Alice's LTC balance

```
 ./litecoin-cli -rpcuser=Alice -rpcpassword=123 getbalance
```

#### Bob's LTC balance

```
 ./litecoin-cli -rpcuser=Bob -rpcpassword=123 getbalance
```

### Initialize Atomic Swap settings

To configure the Atomic Swap settings, enter the following command:

```
 ./beam-wallet set_swap_settings --swap_coin=ltc --swap_wallet_addr= --swap_wallet_user= --swap_wallet_pass= --swap_feerate=<Litecoin fee rate(Photons/Kb)> --active_connection=core
```

{% hint style="info" %}
**Each cryptocurrency has its own transaction fees**.

Avoid transaction jamming or failure by double-checking the`--swap_feerate`for each currency before initiating an Atomic Swap.

LTC, for example, has a`--swap_feerate`per 1KB of transaction size. LiteCoin's`--swap_amount`is denominated in "photons." 1 LTC = 1000000 photons, which is denominated in Beams while also offering fixed fee rates, regardless of transaction size.
{% endhint %}

#### Configuring swap settings for Alice

```
 ./beam-wallet set_swap_settings --swap_coin=ltc --swap_wallet_addr=127.0.0.1:13300 --swap_wallet_user=Alice --swap_wallet_pass=123 --swap_feerate=90000 --active_connection=core
```

#### Configuring swap settings for Bob

```
 ./beam-wallet set_swap_settings --swap_coin=ltc --swap_wallet_addr=127.0.0.1:13400 --swap_wallet_user=Bob --swap_wallet_pass=123 --swap_feerate=90000 --active_connection=core
```

### Swapping LTC for Beam

Both parties can initiate the Atomic Swap by entering the following command:

```
 ./beam-wallet swap_init -n  --amount= --swap_coin=ltc --swap_amount= --swap_beam_side
```

The parameter`--swap_beam_side` is used when the other party (Alice) swaps Beam for a different cryptocurrency.

Bob enters the following command:

```
 -./beam-wallet swap_init -n "eu-node01.mainnet.beam.mw:8100" --amount=10 --swap_coin=ltc --swap_amount=2000000000
```

Bob must copy the swap token and forward it to Alice. After, Bob can enter the`--wallet_listen`command:

```
./beam-wallet listen -n "eu-node01.mainnet.beam.mw:8100"
```

Alice can accept the swap by entering the following command:

```
 ./beam-wallet swap_accept -n "eu-node01.mainnet.beam.mw:8100" --swap_token=316sveQtJrhxzuBy2zJHTp8aHfPgdz2FycrR8n9fs5CbXqoq1Be4Z9qEPnz5HjxuBZgmQpxWd8Dy9icQYKVn1e23cP7x5FHcteyEXk11QQ6CQLQJ3ERk653xgzXnBNfiiX8Pw8acyuNqCHPsF699oiDkxgEAXtV5mrKmYWh1zW
```

### Accepting swap offer

If Alice accepts the swap, it will create a swap transaction, and the LTC and Beam will lock on Bob's blockchain.

{% hint style="info" %}
**Each blockchain has its own block generation time**. Average block generation times (roughly):

* **LiteCoin**: two and a half (2.5) minutes.
* **Beam**: one minute.
* **Bitcoin**: ten minutes.
{% endhint %}

After six blocks, Bob will redeem Alice's Beam coins, revealing a secret transaction code and completing Bob's portion of the Atomic Swap. ‌

Alice will redeem Bob's LTCs using the secret transaction code revealed to her, completing the Atomic Swap.

Bob and Alice can return to their respective Beam and LTC wallets to confirm the details of the Atomic Swap.

## Perform an Atomic Swap with Bitcoin

Using Alice and Bob as an example, we will go over how to perform an Atomic Swap of Alice's Beam coins for Bob's Bitcoin (BTC).

### Run a full BTC node

Bob and Alice must configure their BTC node to allow RPC access either through the command line or edit their config files according to the [BTC documentation](https://en.bitcoin.it/wiki/Running\_Bitcoin) (for this example, we will use standard node and RPC ports).

Alice and Bob can run a full BTC node by entering the following command:

#### Alice's BTC node

```
./bitcoind -server -datadir="Alice/path_to_litecoin_wallet_data" -rpcuser=Alice -rpcpassword=123 -printtoconsole
```

#### Bob's BTC node

```
./bitcoind -server -datadir="Bob/path_to_litecoin_wallet_data" -rpcuser=Bob -rpcpassword=123 -printtoconsole
```

{% hint style="danger" %}
#### The nodes must be synced to the current blockchain height before an Atomic Swap can occur.
{% endhint %}

### Run a full Beam node

Please refer to this [guide](https://beamx.gitbook.io/cli-guide/#synching-with-a-specific-node) for setting up a full Beam node.

Alice and Bob can check their BTC balance by entering the following command

#### Alice's BTC balance

```
./bitcoin-cli -rpcuser=Alice -rpcpassword=123 getbalance
```

#### Bob's BTC balance

```
./bitcoin-cli -rpcuser=Bob -rpcpassword=123 getbalance
```

### Initialize Atomic Swap settings

To configure the Atomic Swap settings, enter the following command:

```
./beam-wallet set_swap_settings --swap_coin=btc --swap_wallet_addr= --swap_wallet_user= --swap_wallet_pass= --swap_feerate=<Bitcoin fee rate(Satoshs/Kb)> --active_connection=core
```

{% hint style="info" %}
**Each cryptocurrency has its own transaction fees**.

Avoid transaction jamming or failure by double-checking the`--swap_feerate`for each currency before initiating an Atomic Swap.

BTC, for example, has a`--swap_feerate`per 1KB of transaction size. Bitcoin's`--swap_amount`is denominated in "satoshis." 1 BTC = 1000000 satoshis, compared to Beam, which is denominated in Beams while also offering fixed fee rates, regardless of transaction size.
{% endhint %}

#### Configuring swap settings for Alice

```
./beam-wallet set_swap_settings --swap_coin=btc --swap_wallet_addr=127.0.0.1:13300 --swap_wallet_user=Alice --swap_wallet_pass=123 --swap_feerate=90000 --active_connection=core
```

#### Configuring swap settings for Bob

```
./beam-wallet set_swap_settings --swap_coin=ltc --swap_wallet_addr=127.0.0.1:13400 --swap_wallet_user=Bob --swap_wallet_pass=123 --swap_feerate=90000 --active_connection=core
```

### Swapping BTC for Beam

Both parties can initiate the Atomic Swap by entering the following command:

```
./beam-wallet swap_init -n  --amount= --swap_coin=btc --swap_amount= --swap_beam_side
```

The parameter`--swap_beam_side`is used when the other party (Alice) swaps Beam for a different cryptocurrency.

Bob enters the following command:

```
./beam-wallet swap_init -n "eu-node01.mainnet.beam.mw:8100" --amount=10 --swap_coin=btc --swap_amount=100000000
```

Bob must copy the swap token and forward it to Alice. After, Bob can enter the`--wallet_listen`command:

```
./beam-wallet listen -n "eu-node01.mainnet.beam.mw:8100"
```

Alice can accept the swap by entering the following command:

```
 ./beam-wallet swap_accept -n "eu-node01.mainnet.beam.mw:8100" --swap_token=316w4oB5hCz2qeVNrtteAEZXhxxx2HBX8v1Ped1FhveJor5JbChz2xXGfi2LkKqVLu8kU4vEoZCV3UbmwoBZX2ABJzmbxLPxpCTVZr1oefwsJDzYU2BUXXDTf4VjtBJfsP3yrozPT4bz1ZTdDTzRS2yU3VYvnamuSRSfEPatha
```

### Accepting swap offer

If Alice accepts the swap, it will create a swap transaction, and the BTC and Beam will lock on Bob's blockchain.

{% hint style="info" %}
**Each blockchain has its own block generation time**. Average block generation times (roughly):

* **LiteCoin**: two and a half (2.5) minutes.
* **Beam**: one minute.
* **Bitcoin**: ten minutes.
{% endhint %}

After six blocks, Bob will redeem Alice's Beam coins, revealing a secret transaction code and completing Bob's portion of the Atomic Swap. ‌

Alice will redeem Bob's BTCs using the secret transaction code revealed to her, completing the Atomic Swap.

Bob and Alice can return to their respective Beam and BTC wallets to confirm the details of the Atomic Swap.
