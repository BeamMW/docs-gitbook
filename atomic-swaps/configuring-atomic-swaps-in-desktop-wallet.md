# Configuring Atomic Swaps In Desktop Wallet

Currently, Atomic Swaps is Desktop Only. For more updates on upcoming Beam news, see our [Telegram page](https://t.me/BeamPrivacy).

https://www.youtube.com/watch?v=_SOgPybgi8Y&t=504s

## Configuring Atomic Swaps

Configure your Beam wallet to enable Atomic Swaps by connecting to the node and wallet of the currency you wish to swap. Each currency can link with your Beam wallet via RPC full node or an Electrum wallet.

## **Connecting to Full Node via Litecoin**

To connect Litecoin with your Beam wallet via full node, follow the steps below:

### **Edit Litecoin Configuration File**

Using a text editor, open the `Litecoin.conf` file and enter the following parameters to configure Litecoin to run with JSON RPC:

```
# server=1 tells Litecoin-QT to accept JSON-RPC commands.
server=1

rpcport=9432

# You must set rpcuser and rpcpassword to secure the JSON-RPC api
rpcuser=liteuser
rpcpassword=123
```

Save your changes to `Litecoin.conf`.

### Synch Litecoin Node with Beam Wallet

Go to **Settings**<img src=".gitbook/assets/Screen Shot 2021-07-03 at 5.06.40 PM.png" alt="" data-size="line">.

![](.gitbook/assets/Screenshot\_203.png)

### Enter Litecoin Node Address

Under the Litecoin tab, **enter the node ip:port address along with the RPC username and password**. The fee paid on the respective chain per Kb of transaction size can be left at the default value.

**Click Apply > Connect.**

A green light will appear next to **Litecoin Node,** indicating the connection was a success!

![](<.gitbook/assets/Screen Shot 2021-05-15 at 2.05.52 PM.png>)

## Connect with Electrum Wallet

If you have an Electrum wallet, connecting to a remote node follows a similar yet more straightforward process:

### Edit Electrum Wallet Settings

1. **Settings**<img src=".gitbook/assets/Screen Shot 2021-07-03 at 5.06.40 PM.png" alt="" data-size="line">**> toggle "Node" to "Electrum."**
2. Enter your Electrum address.
3. Enter an existing seed phrase or generate a new one (you can change wallets at any time).

Once connected, you should be able to see the list of all wallet addresses in your Electrum wallet by clicking "Show wallet addresses."\


![](.gitbook/assets/Screenshot\_206.png)

Once you have optimized and synchronized your nodes in the settings tabs, you will now be able to accept Atomic Swap offers.

![](.gitbook/assets/Screenshot\_207.png)

## Connecting to Ethereum

To link your Ethereum and Beam wallets together, you must have an Infura account as well as an Ethereum Metamask.

[Infura](https://infura.io/) is the API service that broadcasts information from the Ethereum blockchain, and [Metamask](https://metamask.io/) is a browser-based Ethereum wallet.

Note: [infura.io](https://www.infura.io) requires specific parameters from your Beam wallet as well as valid email address verification.


Before attempting to create an Ethereum <> Beam Atomic Swap make sure your wallet balance has enough Ethereum to cover mining fees.


## To connect to Ethereum

1. **Launch** [infura.io.](http://www.infura.io)
2. Go to **Settings > "keys" > Copy Project ID.**
3. **Launch** your Beam wallet.
4. Go to **Settings**<img src=".gitbook/assets/Screen Shot 2021-07-03 at 5.06.40 PM.png" alt="" data-size="line">**> "Connectivity" > "Ethereum" > Paste Project ID.**
5. **Click "Generate Seed Phrase"** to generate the seed phrase you will import to your Metamask. You can also use an existing seed phrase.
6. **Copy seed phrase > "apply changes" > "Connect to node."**
7. **Launch** **Metamask > click "import using account seed phrase."**
8. Enter your seed phrase and create a strong wallet password.

If you have successfully linked your accounts, a green light will appear next to the Ethereum node indicating the connection was a success!

## **How it looks in your wallet**

### Infura.io

Copy this project ID string.

![](.gitbook/assets/Screenshot\_172.png)

### Beam wallet settings

Paste your Infura Preject ID under "Ethereum."

![](.gitbook/assets/Screenshot\_173.png)

### Generate a new seed phrase

Click "apply changes" after you have copied your seed phrase.

![](.gitbook/assets/Screenshot\_174.png)

### Connect to Ethereum

![](.gitbook/assets/Screenshot\_175.png)

### Import Ethereum seed phrase

The next step is to import the Ethereum seed phrase into a Metamask wallet. At the bottom of your Metamask browser extension, click "Import using account seed phrase." Enter your seed phrase and create a password.

![](.gitbook/assets/Screenshot\_176.png)

### Atomic Swaps screen

![](.gitbook/assets/Screenshot\_179.png)
