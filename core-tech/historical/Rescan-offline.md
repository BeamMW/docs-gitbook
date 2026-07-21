> **Historical Document** — This document describes an offline rescan procedure for earlier wallet versions. It is preserved for historical reference only.

# Rescan offline
This scenario is for cases when you don't want to stop running node and wallet and you wish to ensure if the balance is correct.

1. Create a clone of your running wallet. This can be done by restoring the wallet from the seed

`./beam_wallet restore --seed_phrase=<a;d;b>`

or

 by copying of its `wallet.db` file

It can be done on separate machine with no internet connection.

2. Copy your running node.db. to these machine

3. Ensure that this copied node has *your* owner_key. Export it from your cloned wallet

`./beam_wallet export_owner_key`

and launch your node 

`./beam_node --owner_key=<key>`

Note that this node may have no connection to internet, as a result this wallet will not accept incoming transactions.

`Node.db` contains the blockchain data up to the height when you did a copy of it.
Now, with owner key node should distinguish all your transaction outputs. 
 

4. Connect you cloned wallet to your offline node

`./beam_wallet listen -n 127.0.0.1:10000`

or if you copied existing wallet.db 

`./beam_wallet rescan -n 127.0.0.1:10000