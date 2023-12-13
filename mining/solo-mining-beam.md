# Solo Mining Beam

Mining Beam coins is the best way to support Beam! Miners strengthen the security of our network while earning Beam coins. In this guide, we will provide you all the information needed to start mining solo mining Beam!

## Things you'll need

* Mining rig containing a minimum of one GPU with 3GB of RAM.
* Basic knowledge of command line (CMD terminal) functions.
* Text editor, e.g., [Visual Code Studio](https://code.visualstudio.com/).
* High-speed internet connection.
* Mining software, e.g., [lolMiner](https://github.com/Lolliedieb/lolMiner-releases).
* An 'always-on PC' or a similar device to run your Beam Node.

## **Part one: installation**

Download `Beam-Wallet-CLI` and `Beam-Node` files directly from our main [website](/downloads).

‌Extract these files in a separate folder on your 'always-on PC' (or whichever device is running your Node) and label the folder as `beam-mining`.

## **Part two: setup**

### **Create SSL Certificate and Key**

To create an SSL Certificate and Key within `beam-mining`, open a new CMD terminal window. Enter the following command:

```
openssl req -x509 -newkey rsa:4096 -keyout beam-stratum-key.pem -out beam-stratum-crt.pem -days 3650 -nodes -subj '/CN=localhost’
```

Rename the SSL Certificate and Key files created by the output.

The output should be similar to the following:

```
mv beam-stratum-key.pem stratum.key
mv beam-stratum-crt.pem stratum.crt
```

Open your `beam-wallet.cfg` file in a text editor. Remove the `#` before `pass=password` and `node_addr=` . Enter your password and node address.

The output should be similar to the following:

```
pass=yourpasswordhere
node_addr=127.0.0.1:10000
```

Save the changes made to your `beam-wallet.cfg`file.

### Initialize your Beam wallet

To initialize your wallet and create a seed phrase, enter the following command:

```
./beam-wallet init
```

Save your seed phrase (do not take a screenshot) and store it somewhere safe. Pen and paper is recommended. Do not store your seed phrase electronically or anywhere vulnerable to hackers.

### Exporting your miner key

To export your miner key, enter the following command:

```
./beam-wallet export_miner_key --subkey=1
```

Copy the key string output. Using your text editor, open `beam-node.cfg` and paste the key string output after `miner_key=`.

### Exporting your owner key

To export your owner key, enter the following command:

```
./beam-wallet export_owner_key
```

Copy and paste the key string output in your `beam-node.cfg` file after `owner_key=`.

Copy your password from your `beam-wallet.cfg` file and then paste it into your `beam-node.cfg` after `pass=`.

### Enter Beam node parameters

Add the following parameters (one per line) to your `beam-node.cfg` file:

```
port=10000
log_level=verbose
file_log_level=verbose
peer=eu-nodes.mainnet.beam.mw:8100,us-nodes.mainnet.beam.mw:8100,ap-nodes.mainnet.beam.mw:8100,ap-hk-nodes.mainnet.beam.mw:8100,shanghai-node.mainnet.beam.mw:8100
stratum_port=3333
stratum_secrets_path=.
```

Save the changes made to your `beam-node.cfg` file.

### Join the Beam network

To join the Beam network, open a new CMD terminal window. Enter the following command:

```
./beam-node
```

Leave the CMD terminal window open to keep the node running as you wait for the node to sync with the Beam network (this may take a few hours).

After your node syncs with the network, open a separate CMD terminal (do not close node CMD terminal window), and enter the following command:

```
./beam-wallet listen
```

Leave this terminal open to keep the wallet listening to your node.

### Launch mining software

Your mining rig can use your node's IP address and stratum port to begin solo mining Beam!

Using [lolMiner](https://github.com/Lolliedieb/lolMiner-releases) as an example, the command output is similar to the following:

```
./lolMiner --coin BEAM --pool 127.0.0.1:3333 --user yourwalletaddress
```

After you mine a block, your block rewards will show up in your wallet summary.

To view your wallet summary, enter the following command:

```
./beam-wallet info
```

The output is similar to the following:

![example wallet summary after solo mining Beams](<.gitbook/assets/Screenshot 2021-07-06 at 12.44.19.png>)
