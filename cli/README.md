---
description: The "official" Beam command line reference guide!
---

# Beam command line (CLI) guide

The command line is the most reliable method of using Beam wallets and certain features (Registering CAs, Lightning Payments, Beam Shaders) can only perform on the command line. Available for macOS, Windows, and Linux.

Due to Beam's LelantusMW protocols, the private keys of Beam wallets record the user's UTXO balance on the blockchain, while transaction histories, addresses, and other wallet metadata are stored locally on each device.

Beam allows users to run a full node and interact with it via the command line.

## Getting started

This section uses Linux and macOS commands; Windows users, please substitute`./beam-wallet`with `beam-wallet.exe` and Windows Command Prompts where necessary.

## Creating a new Beam wallet

To create a new wallet, enter the following command:

```
./beam-wallet init
```

The`-wallet init` command will ask users to create a wallet password:

```
Enter password: ****************
```

Create a strong wallet password and keep your password secret! Anyone who has your wallet password will have access to your funds! Wallet passwords protect your `wallet.db` files that store all your crucial wallet metadata.

After creating your wallet password, the`-wallet init`command output should resemble the following:

```
$ ./beam-wallet init
I 2018-12-23.15:24:29.461 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.15:24:29.462 starting a wallet...
Enter password: ****************
I 2018-12-23.15:24:32.524 Generating seed phrase...

Generated seed phrase:

        despair;evoke;airport;seven;cricket;menu;current;ankle;require;monkey;maple;crawl;

        IMPORTANT

        Your seed phrase is the access key to all the cryptocurrencies in your wallet.
        Print or write down the phrase to keep it in a safe or in a locked vault.
        Without the phrase you will not be able to recover your money.

I 2018-12-23.15:24:32.728 wallet successfully created...
I 2018-12-23.15:24:32.750 New address generated:

14a38140d8e66be9b8f1e8d770161fd33e35f7000053147b5a0f6a83178926b956

I 2018-12-23.15:24:32.750 label = default
```

The`rules signature`is a hash of the current node configuration that determines compatibility between different versions of nodes and wallets.

{% hint style="danger" %}
**Write down your seed phrase!** Keep your seed phrase secret, and never share your seed phrase with anyone!
{% endhint %}

Your seed phrase is the only way you (or anybody) can access your funds. Your seed phrase is not linked to your email or phone number. If you forget your wallet password or lose your device, the only way to restore your wallet is through your seed phrase!&#x20;

Please do not write your seed phrase down electronically, take a screenshot or store it anywhere vulnerable to cyber attacks. Pen and paper are unhackable and recommended. Make multiple copies of your seed phrase and store them somewhere safe.

## Restoring CLI wallet from seed phrase

Restoring Beam coins from an old wallet seed phrase requires users to run their own node and [`owner_key`](./#exporting-owner-key).

To begin restoring your CLI wallet, first create a new wallet by entering the following command:

```
./beam-wallet restore --seed_phrase=<semicolon separated list of 12 seed phrase words>;
```

Now export your `owner_key` from your Beam wallet. Enter the following command:

```
./beam-wallet export_owner_key
```

Now that you have exported the `owner_key`, you can run your own node by entering the following command:

```
./beam-node --peer=<ip and port of peer node> --key_owner=<owner key exported from the wallet>
```

Once the node has synchronized, connect your wallet to the node and update your wallet database by entering the following command:

```
./beam-wallet listen -n <ip and port of your node, ex:127.0.0.1:10000>
```

Once your wallet has synched with the Beam network, enter `-wallet info` command to check the wallet status:

```
./beam-wallet info
```

## Exporting secret keys

### Exporting miner key

The`miner_key`is a secret key that miners can use to collect their miner rewards.&#x20;

To export your`miner_key`, enter the following command:

```
./beam-wallet export_miner_key --subkey=
```

Enter your wallet password.

The output should resemble the following:

```
$ beam-wallet.exe  export_miner_key --subkey=1
I 2018-12-23.16:36:04.306 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.16:36:04.307 starting a wallet...
Enter password: *******************
Secret Subkey 1: OVBSdWQlOV3WuC6bLXRDJqyDfdxWSuzdA4jEGRAZ1zhy4gA3/KcBTEdcmN5wNOv0vQrBWwOlTdIxqyPFzFDFdaVYZPUDoXjqgUE=
```

**Keep your`miner_key`secret!** Anyone with the`miner_key`can retrieve the Miner Rewards associated with said keys.&#x20;

### Exporting owner key

An`owner_key`connects all the nodes mining under that key and communicates with all the other nodes on the network of accumulated miner rewards. If a scenario occurs where a user has multiple mining nodes under one`owner_key`, the user would only need to connect to a single node to collect the miner rewards.&#x20;

Most cryptocurrencies allow users to connect mining nodes to a single address; however, the Mimblewimble protocol has no addresses. Unique blinding factors are deterministically derivative from the`master_key`code into the miner rewards tagged by a single`owner_key`.

While an`owner_key`cannot access users' funds, it allows users to view all the coins mined by every mining node associated with said key. Keep your`owner_key`secret.

To export`owner_key`from your Beam wallet, enter the following command:

```
./beam-wallet export_owner_key
```

Enter your wallet password.

Sample output for this command should look like this:

```
$ ./beam-wallet export_owner_key
I 2018-12-23.16:53:04.973 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.16:53:04.974 starting a wallet...
Enter password: *
Owner Viewer key: dmVxtRCM3BH1VakviSB/XY86DsCKuWDLKk51eLDlibgMeL2fZ317Zdqx3
```

## Printing wallet info

Printing your wallet info will give you a snapshot of your wallet, including wallet balance, block height, current state ID, available balance, total groth, and more.

To print the current status of your wallet, enter the following command:

```
./beam-wallet info
```

Enter your wallet password.

The output should resemble the following:

```
I 2018-12-23.17:56:19.368 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.17:56:19.369 starting a wallet...
Enter password: *
I 2018-12-23.17:56:21.144 wallet sucessfully opened...
____Wallet summary____

Current height............8353
Current state ID..........72329a2efa2ddad4

Available.................300 beams
Maturing..................0 groth
In progress...............0 groth
Unavailable...............0 groth
Available coinbase .......0 groth
Total coinbase............0 groth
Avaliable fee.............0 groth
Total fee.................0 groth
Total unspent.............300 beams

                  id |          Beam |         Groth |        height |          maturity |                  status |    type
    1545571472000001             300               0            8347                8351   [Available]                 norm
```

### Transaction history

Users can view their transaction history with the`--tx_history` tag:

```
./beam-wallet  info --tx_history
```

### View transaction details

To view transaction details, add the`--tx_id`tag:

```
./beam-wallet tx_details --tx_id=
```

## Receiving Beam

### Synching with a specific node

Users must connect to a specific node before they can start receiving Beam coins.&#x20;

To connect to a specific node, enter the following command:

```
./beam-wallet listen -n 
```

Enter your wallet password.

The output should resemble the following:

```
I 2018-12-23.17:07:55.526 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.17:07:55.527 starting a wallet...
Enter password: ***************
I 2018-12-23.17:07:58.076 wallet sucessfully opened...
I 2018-12-23.17:07:58.078 WalletID 14a38140d8e66be9b8f1e8d770161fd33e35f7000053147b5a0f6a83178926b956 subscribes to BBS channel 20
I 2018-12-23.17:07:59.297 Sync up to 8304-2dc4e5a393d6774b
I 2018-12-23.17:07:59.318 Current state is 8304-2dc4e5a393d6774b
```

Once your wallet syncs with the specific node, your wallet will automatically scan the network for incoming transactions via SBBS addresses.

### Receiving Beam coins

To Receive Beam coins, send your wallet address to the sender over a secure channel such as email or Telegram.

All incoming transactions will be viewable in your wallet logs and command terminal.

The output should be similar to the following:

```
I 2018-12-23.17:55:08.556 [7997ecd5c59e4865a6d938dbf339567e] Receiving 300 beams  (fee: 10 groth )
I 2018-12-23.17:55:08.608 [7997ecd5c59e4865a6d938dbf339567e] Invitation accepted
D 2018-12-23.17:55:09.203 Received PeerSig:     596857beae016ebd
I 2018-12-23.17:55:09.216 [7997ecd5c59e4865a6d938dbf339567e] Transaction kernel: 95a8e48587c452b3
D 2018-12-23.17:55:09.346 [7997ecd5c59e4865a6d938dbf339567e] has registered
D 2018-12-23.17:55:09.367 Received PeerSig:     596857beae016ebd
I 2018-12-23.17:55:09.428 Get proof for kernel: 95a8e48587c452b3
```

## Sending Beam

{% hint style="info" %}
**Things to consider:**

1. One Groth is equal to $$1e-8$$Beam.
2. You can split a large UTXO by creating a transaction and sending a portion of the UTXO to yourself. Use the`-wallet send`command to send the required amount to your own SBBS wallet address. A transaction fee will still be required.
{% endhint %}

To Send Beam coins, enter the following command:

```
./beam-wallet send -n  -r  -a  -f 
```

The output should resemble the following:

```
$ ./beam-wallet send -n 172.104.249.212:8101 -r 14a38140d8e66be9b8f1e8d770161fd33e35f7000053147b5a0f6a83178926b956 -a 10
I 2018-12-23.18:05:49.037 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.18:05:49.038 starting a wallet...
Enter password: *
I 2018-12-23.18:05:50.725 wallet sucessfully opened...
I 2018-12-23.18:05:50.726 WalletID 14a38140d8e66be9b8f1e8d770161fd33e35f7000053147b5a0f6a83178926b956 subscribes to BBS channel 20
I 2018-12-23.18:05:50.775 [b21f08337dd94603bb038c82c1888eac] Sending 10 beams  (fee: 0 groth )
I 2018-12-23.18:05:50.986 [b21f08337dd94603bb038c82c1888eac] Invitation accepted
I 2018-12-23.18:05:51.053 [b21f08337dd94603bb038c82c1888eac] Transaction kernel: 71cf20c4c94f25ce
```

## Sending a specific UTXO

For certain transactions, the user might consider sending a specific UTXO.&#x20;

### Print wallet info

To find a specific UTXO,  enter the`-wallet info`command:

```
./beam-wallet info
```

### Sending UTXO

Review and select the UTXO you wish to send.

Enter the `-wallet send`command with a`--utxo`tag, (use a comma to separate multiple UTXO IDs):

```
./beam-wallet send -n  -r  -a  -f  --utxo=
```

## Cancel and delete a transaction

There might be an instance where it is necessary to cancel or delete an unsuccesful transaction.

### Canceling a transaction

First, print your wallets transaction history using `--tx_history` command:

```
./beam-wallet  info --tx_history
```

After locating the`--tx_id`t you wish to cancel, enter the following command:

```
./beam-wallet cancel_tx --tx_id= -n 
```

### Deleting a transaction

To delete a canceled transaction, enter the following command:

```
./beam-wallet delete_tx --tx_id= -n 
```

## Create a new wallet address

Due to Beam's [LelantusMW protocol](https://app.gitbook.com/@beamx/s/developer-documentation/beam-technology/lelantusmw), your wallet address will never record on the blockchain. Beam wallets automatically create a new wallet address for every transaction. Beam offers users three different wallet address types; read about the different address types [here](https://beam.mw/fr/docs/ecosystem#beam-addresses).

### Create a regular address

To create a regular address, enter the following command:

```
./beam-wallet get_address --expiration_time=never --comment="some comment"
```

Enter your wallet password.

Sample output should be similar to the following:

```
I 2021-05-27.22:25:46.490 Rules signature: 0-6bf65af9395e4f40, 30-5d3bc9e672f2b4fb, 30-690e516d8f3dc78c, 1500-b14773a6c55533e5
Enter password: *
I 2021-05-27.22:25:47.892 wallet successfully opened...
I 2021-05-27.22:25:47.903 New WalletID (SBBS address) generated: 135ae78e4b06622c535614494c6feda7de02376f9eb86bef4cd0be8d41a1af3419f
I 2021-05-27.22:25:47.904 Generated regular new style address:8NmCQLhTTk2P7XeFZrHe1SNv7CMEZVsZsKesnpu615wiotYpwDnTDE57pAGTTYM4yD8JThkeLpLLVeYoVuZMZxS5cp6GHaTn16eCBKpZfkK3jDMhPjUMPZpwBgmzpStvDQweLY9ALcXAh
New address: 8NmCQLhTTk2P7XeFZrHe1SNv7CMEZVsZsKesnpu615wiotYpwDnTDE57pAGTTYM4yD8JThkeLpLLVeYoVuZMZxS5cp6GHaTn16eCBKpZfkK3jDMhPjUMPZpwBgmzpStvDQweLY9ALcXAh
```

### Create an SBBS address

To create a new SBBS address, enter the following command:

```
./beam-wallet new_addr --expiration_time=auto|never|now --comment="some comment"
```

Enter your wallet password.

Sample output should be similar to following:

```
I 2018-12-23.18:16:44.112 Rules signature: ddccf5d8d0f77bd2
I 2018-12-23.18:16:44.113 starting a wallet...
Enter password: *
I 2018-12-23.18:16:45.392 New address generated:

646a773da4d4651f35fd75ca958b7859e89d8d8382b8155773bd396e2cc49cca
```

### Create an Offline address

{% hint style="info" %}
It is essential to provide the number of payments (i.e., vouchers). If the payments reach zero, the Sender wallet will automatically request more payments to the Receiver wallet using SBBS. If the Receiver's wallet is online within 12 hours of the request (until the SBBS message expires), it will send 30 more offline payment vouchers to the Sender. Otherwise, the Sender will receive notification that there are no more vouchers and need to request another offline address via an external channel.
{% endhint %}

To create an Offline address, enter the following command:

```
./beam-wallet get_address --offline_count 1
```

Enter your wallet password.

The output should be similar to following:

```
I 2021-05-27.22:26:24.206 Rules signature: 0-6bf65af9395e4f40, 30-5d3bc9e672f2b4fb, 30-690e516d8f3dc78c, 1500-b14773a6c55533e5
Enter password: *
I 2021-05-27.22:26:25.494 wallet successfully opened...
I 2021-05-27.22:26:25.497 New WalletID (SBBS address) generated: 21735e5ecb29171be175d5d1714090412789b71e82dbe8d7aeedb38160c7349bd0b
I 2021-05-27.22:26:25.500 Generated offline address: vn9cQCXgkE9UaCYbbtpwXdkmj9k7EycL8RHkcyCxkMij9rJduzUarm1w4jppqyoF29VxHviS8RhfY4vXfR5ntKv5aKeK4NsRW7xfZhPb9dVaLwKG26MoFAq4qD71kQ5ie1aN16UZw63QJ4N4Z6JydbevHG55t7T2k1HXbfgP5qgGxUAqWSHsQzMk9h1AmLwNXwENygE58idNoG6kcfX7NBcpspcik9m334ZCwb1Ura6UFF7XRhZpLmaReYkqBENzin6tDYfFPvZyGGpGMxyhCNtDTtKSfeif3zvfLmJJ3Qx3hxpyw8tL1R3hSXWS78hpA2ye3qiFpRdRM2RtR2bjoL8EvRdHToQoChLP9YGw1fQhqC8bBqtcNKKCFwQDycyuBQHWi6y9UiegzDKiFrCDSHABVLB7JBGNudcNQNk8x8tggqWjWWooRa3epM8SKQrbEeMRVEaHiK5Tco4wt4pqRyK, vouchers count 1
New address: vn9cQCXgkE9UaCYbbtpwXdkmj9k7EycL8RHkcyCxkMij9rJduzUarm1w4jppqyoF29VxHviS8RhfY4vXfR5ntKv5aKeK4NsRW7xfZhPb9dVaLwKG26MoFAq4qD71kQ5ie1aN16UZw63QJ4N4Z6JydbevHG55t7T2k1HXbfgP5qgGxUAqWSHsQzMk9h1AmLwNXwENygE58idNoG6kcfX7NBcpspcik9m334ZCwb1Ura6UFF7XRhZpLmaReYkqBENzin6tDYfFPvZyGGpGMxyhCNtDTtKSfeif3zvfLmJJ3Qx3hxpyw8tL1R3hSXWS78hpA2ye3qiFpRdRM2RtR2bjoL8EvRdHToQoChLP9YGw1fQhqC8bBqtcNKKCFwQDycyuBQHWi6y9UiegzDKiFrCDSHABVLB7JBGNudcNQNk8x8tggqWjWWooRa3epM8SKQrbEeMRVEaHiK5Tco4wt4pqRyK
```

### Locate Public-Offline address

{% hint style="info" %}
A Public-Offline address is unique address to your Beam wallet and never expires. Public Offline addresses are a little less secure in one aspect: the Sender can potentially see when the Receiver spends the funds sent to this address (though, of course, not to whom or how much). We recommend using this address for donations or a Receiver wallet that is rarely online.
{% endhint %}

To locate your Beam wallet's Public-Offline address, enter the following command:

```
./beam-wallet get_address --public_offline
```

Enter your wallet password

The output should be similar to the following:

```
I 2021-05-27.22:27:13.925 Rules signature: 0-6bf65af9395e4f40, 30-5d3bc9e672f2b4fb, 30-690e516d8f3dc78c, 1500-b14773a6c55533e5
Enter password: *
I 2021-05-27.22:27:14.837 wallet successfully opened...
I 2021-05-27.22:27:14.841 New WalletID (SBBS address) generated: 276d2fecb6aa4f6b2766b37c4cf053e4b8c2b4dec2f37d2be2ae6cbfeb910a0aeb1
I 2021-05-27.22:27:14.843 Generated public offline address: A4Y8nZvGgzE7hvGT5LezYpTJA5BqU1FX2ZSfEqsWcJU1dLh8Rpv8cKo7rcJpLcANFDYWv6ik82PoVGCjeoRu9JUccVCLU8t1HBdrT4jLik6vpZs2r1aDWmML3h4THG7LRWM58tJDjAkVEFgHiTePjmTX5QzLNKuT3MpcG2dXVcHU8dG9sBcf9iLx1b117qRwvb7Y19mqkVNb4qdcMPzmtEwV748ZqdXdMB9hZpdRuq1Qk5pBwfSYgtyG2kRdPmSnhggRYK3xvep5BVn6zuZ3LLpe2uo3krrXxhQ4MyQknV
New address: A4Y8nZvGgzE7hvGT5LezYpTJA5BqU1FX2ZSfEqsWcJU1dLh8Rpv8cKo7rcJpLcANFDYWv6ik82PoVGCjeoRu9JUccVCLU8t1HBdrT4jLik6vpZs2r1aDWmML3h4THG7LRWM58tJDjAkVEFgHiTePjmTX5QzLNKuT3MpcG2dXVcHU8dG9sBcf9iLx1b117qRwvb7Y19mqkVNb4qdcMPzmtEwV748ZqdXdMB9hZpdRuq1Qk5pBwfSYgtyG2kRdPmSnhggRYK3xvep5BVn6zuZ3LLpe2uo3krrXxhQ4MyQknV
```

## Print address list&#x20;

To print your Beam wallet address list, enter the following command:

```
./beam-wallet address_list
```

The output should be similar to the following:

```
I 2021-05-27.22:29:03.109 Rules signature: 0-6bf65af9395e4f40, 30-5d3bc9e672f2b4fb, 30-690e516d8f3dc78c, 1500-b14773a6c55533e5
Enter password: *
I 2021-05-27.22:29:04.807 wallet successfully opened...

Type: public offline
Comment:
Address: A4Y8nZvGgzE7hvGT5LezYpTJA5BqU1FX2ZSfEqsWcJU1dLh8Rpv8cKo7rcJpLcANFDYWv6ik82PoVGCjeoRu9JUccVCLU8t1HBdrT4jLik6vpZs2r1aDWmML3h4THG7LRWM58tJDjAkVEFgHiTePjmTX5QzLNKuT3MpcG2dXVcHU8dG9sBcf9iLx1b117qRwvb7Y19mqkVNb4qdcMPzmtEwV748ZqdXdMB9hZpdRuq1Qk5pBwfSYgtyG2kRdPmSnhggRYK3xvep5BVn6zuZ3LLpe2uo3krrXxhQ4MyQknV
Wallet ID: 276d2fecb6aa4f6b2766b37c4cf053e4b8c2b4dec2f37d2be2ae6cbfeb910a0aeb1
Identity: 3e5ca67ca3466595be5a13f722247a888fa5bfb4b84022c155f2cf29c0afb088
Active: true
Expires: 2021.05.28 22:27:14
Created: 2021.05.27 22:27:14

Type: offline
Comment:
Address: vn9cQCXgkE9UaCYbbtpwXdkmj9k7EycL8RHkcyCxkMij9rJduzUarm1w4jppqyoF29VxHviS8RhfY4vXfR5ntKv5aKeK4NsRW7xfZhPb9dVaLwKG26MoFAq4qD71kQ5ie1aN16UZw63QJ4N4Z6JydbevHG55t7T2k1HXbfgP5qgGxUAqWSHsQzMk9h1AmLwNXwENygE58idNoG6kcfX7NBcpspcik9m334ZCwb1Ura6UFF7XRhZpLmaReYkqBENzin6tDYfFPvZyGGpGMxyhCNtDTtKSfeif3zvfLmJJ3Qx3hxpyw8tL1R3hSXWS78hpA2ye3qiFpRdRM2RtR2bjoL8EvRdHToQoChLP9YGw1fQhqC8bBqtcNKKCFwQDycyuBQHWi6y9UiegzDKiFrCDSHABVLB7JBGNudcNQNk8x8tggqWjWWooRa3epM8SKQrbEeMRVEaHiK5Tco4wt4pqRyK
Wallet ID: 21735e5ecb29171be175d5d1714090412789b71e82dbe8d7aeedb38160c7349bd0b
Identity: fa1224adb9cfabc0c062fb14adc17befd1a4a621e01d596952e39aa5b62dfc99
Active: true
Expires: 2021.05.28 22:26:25
Created: 2021.05.27 22:26:25

Type: regular new style
Comment:
Address: 8NmCQLhTTk2P7XeFZrHe1SNv7CMEZVsZsKesnpu615wiotYpwDnTDE57pAGTTYM4yD8JThkeLpLLVeYoVuZMZxS5cp6GHaTn16eCBKpZfkK3jDMhPjUMPZpwBgmzpStvDQweLY9ALcXAh
Wallet ID: 135ae78e4b06622c535614494c6feda7de02376f9eb86bef4cd0be8d41a1af3419f
Identity: 988d4a755bf73bab7da13bf7aeed2d5b3a1f2a767e4d2267daeae61179f1107b
Active: true
Expires: 2021.05.28 22:25:47
Created: 2021.05.27 22:25:47

Type: regular old style
Comment:
Address: f6b11ccea8d22248f6eadbf94bd5741708ff3afbb727a47be9c2da509891047891
Wallet ID: f6b11ccea8d22248f6eadbf94bd5741708ff3afbb727a47be9c2da509891047891
Identity: ce3a1fac23e791642fe93d9f0f38a0644d549cf031f19a7cdbc5138a334f0e44
Active: true
Expires: 2021.07.27 22:24:42
Created: 2021.05.27 22:24:42

```

## Change address expiry date

Beam wallets create a new wallet address for every transaction, It is possible however, to extend the expiration (24 hours) for an old address.

### Extending the expiration date for a single wallet address

To extend your wallet address expiration date, enter the following command:

```
./beam-wallet change_address_expiration --address=14e191aaebace13b14e3ab41382280baff288faa312545eadd1a1bcfa3adaeac6ff
```

### Extending the expiration for all addresses in your Beam wallet

To extend your wallets existing addresses, use the previous command, omitting the `--address`tag.

### Choosing an expiration date

Wallet addresses provided users with three different expiration date options:

* **Never**: address expiration date will never expire.
* **Auto**: wallet address expiration date will extend for an additional 24 hours before expiring.
* **Now**: the wallet address will be expire immediately and will no longer be usable.

Enter the following command:

```
./beam-wallet change_address_expiration --address=<sbbs address> --expiration_time=(never|now|auto)
```

## Export and import wallet metadata

If you have previously created a Beam wallet, you can recover your funds by restoring your wallet from your 12-word seed phrase. However, your wallet data (transaction history, contacts, addresses) is always stored locally onto your`wallet.db`file.&#x20;

### Exporting wallet metadata

To export your`wallet.db`file, enter the following command:

```
./beam-wallet export_data --file_location=<full path to addresses file, for example: C:\Users\user\addresses.dat>
```

### Importing wallet metada

To import your`wallet.db`file, enter the following command:

```
./beam-wallet import_data --file_location=<full path to addresses file, for example: C:\Users\user\addresses.dat>
```

## Proof of transaction

Since`Beam 1.1.4194`, the Receiver wallet has automatically signed proof of received transaction before sending it to the Sender. Upon request, the sender can generate proof of transaction.

{% hint style="info" %}
When Sending Beam coins, adding the`–payment_proof_required=1`tag will require the Receiver to send the payment proof to Sender.&#x20;
{% endhint %}

### Print transactions list

Locate the`--tx_id`. Enter the following command:

```
./beam-wallet  info --tx_history
```

### Get payment proof

After locating the`--tx_id`, enter the following command:

```
./beam-wallet payment_proof_export --tx_id=<txid>
```

The output should resemble the following:

```
I 2019-01-14.14:40:37.464 Payment tx details:
Sender: 4bd0ca080bd8c3ec4b3061bf5916aa34266f0649a7c151c6777ffe492f15e09768
Receiver: ebb27b5501213c84eb212ea276e8ced74f540fbcceb0f4c1c2da2c5108188651a1
Amount: 6 groth
KernelID: 4ac2f195ce9056c171fd0cd41e8a02dc9c0bb72861b2e03fbbbb5942e5e63d1a

I 2019-01-14.14:40:37.465 Sender address own ID: 1547460707000004
I 2019-01-14.14:40:37.465 Exported form: 000000000000004bd0ca080bd8c3ec4b3061bf5
916aa34266f0649a7c151c6777ffe492f15e0976800000000000000ebb27b5501213c84eb212ea27
6e8ced74f540fbcceb0f4c1c2da2c5108188651a1864ac2f195ce9056c171fd0cd41e8a02dc9c0bb
72861b2e03fbbbb5942e5e63d1a7728a2954a10d3bfb9938f0c17509a6a0e870c6bb22ff2d1297f3
dae7f54592b00e84c6b3c9ea3e3ad9bc43661b6dcf7dbd818ccc92707d1d75b429697e8492653
```

### Send payment proof

Copy, paste and send the payment proof (`exported form`) to the Receiver.&#x20;

The payment proof (from the above example) should resemble the following:&#x20;

```
I 2019-01-14.14:40:37.465 Exported form: 000000000000004bd0ca080bd8c3ec4b3061bf5
916aa34266f0649a7c151c6777ffe492f15e0976800000000000000ebb27b5501213c84eb212ea27
6e8ced74f540fbcceb0f4c1c2da2c5108188651a1864ac2f195ce9056c171fd0cd41e8a02dc9c0bb
72861b2e03fbbbb5942e5e63d1a7728a2954a10d3bfb9938f0c17509a6a0e870c6bb22ff2d1297f3
dae7f54592b00e84c6b3c9ea3e3ad9bc43661b6dcf7dbd818ccc92707d1d75b429697e8492653
```

### Verifying the payment proof

The Receiver can verify the payment proof by entering the following command:&#x20;

```
./beam-wallet payment_proof_verify --payment_proof=<proof>
```

## Rescan wallet

Beam wallets regularly monitor the blockchain and sync new information with the wallet. If you suspect your balance, transaction, or UTXO status is not updated, the `-wallet rescan` command will sych your wallet with the blockchain's latest state.

In order to rescan the CLI wallet please follow the steps below:

### Run the node with your **owner key**

Running a node with your [owner key](broken-reference) will verify if your wallet is synchornized with the network.&#x20;

To run node with owner key, enter the following command:

```
./beam-node --peer=<ip or url of the peer> --owner_key=<your owner key>
```

### Rescan the blockchain

To rescan the blockchain, enter following command:

```
./beam-wallet rescan -n <ip:port of the node with the owner key>
```

### Listen for updates

To listen for any updates to the node, enter the`-wallet listen`command:

```
./beam-wallet listen -n <ip:port of the node with the owner key>
```

### Wait for synchronization

After your wallet and node have syncronized, recheck your wallet balance, transaction histories by entering the`-wallet info`command:&#x20;

```
./beam-wallet info
```
