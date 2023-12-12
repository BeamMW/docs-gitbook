# D

## Dandelion

A privacy enhancing transaction routing mechanism, proposed by [Giulia Fanti](https://www.andrew.cmu.edu/user/gfanti/) et al. based on Bitcoin’s BIP-156, that provides anonymity guarantees against deanonymization attacks.

Without it, nodes transmit the transaction to their peers, a process known as diffusion, in a statistically symmetric way. This allows surveillance mechanisms to triangulate the IP location of the transaction, by strategically placing colluding spying nodes around routing locations.

Dandelion mitigates this by sending transaction over a random path before diffusion, in the same way, that dandelion stems fly through the air before spreading its seeds. Transactions travel silently along a path during an anonymous “stem phase,” and are then diffused to the network during the “fluff phase.”

While other privacy solutions aim to protect individual users, Dandelion protects anonymity by limiting the capability of adversaries to deanonymize the entire network.

## Data Directory

The disk location where Beam’s data files are stored, including the wallet data file.

Wallet files will be created in following folders:

- Mac: `/Users/{your_user_name}/Library/Application Support/Beam Wallet/`
- Windows: `\Users\{your_user_name}\AppData\Local\Beam Wallet`
- Linux: `/home/{your_user_name}/.local/share/Beam Wallet`

Upon running a Beam node, log files are located in the logs folder, which resides in the node folder. Once started, the node will create a node.db file in the same folder it is located. This file will store an internal state of the node. Upon the first launch, the node will download current blockchain history in batch mode as a single large macroblock. After the initial sync is complete, the node will continue to sync blocks and individual transactions from the current blockchain Tip (height) and onwards.