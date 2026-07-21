> **Historical Document** — This document describes out-of-sync wallet behaviour for early Beam wallet versions (≤ 2.0.4431 and the subsequent fix). It is preserved for historical reference only.

# Out of sync wallets
This document describes scenarios of transferring beams between wallets which are not in sync with node or syncing is in progress.

Beam is decentralized system, as a result wallet and node cannot be 100% confident about their current state. Is their last known block real tip of the chain or not? - the question wallet and node are trying to resolve through their life.  
In these circumstances, wallets may communicate with each other being on different heights of the chain. In general, this is not a problem if these wallets will come out with a valid transaction which will be accepted by the node. But there are some possibilities to create invalid transaction in this case:
- sender has selected outdated (or spent) UTXO for transaction (possible reasons: wallet clones)
- transaction kernel max height is less than current chain tip height. For this case there are following scenarios:

## Old wallets (<= 2.0.4431)

In this version the sender sets min and max height of the kernel.

- if sender is in sync whereas receiver is out-of-sync the kernel's height will be correct, receiver will simply accept the transaction and sender will send it to the node.

- if sender is out-of-sync and its known tip height is significantly less than current chain's height, then it will create kernel which will be rejected by the node and transaction will be failed.

- if sender is out-of-sync and the difference in height with the chain is less than transaction lifetime (default lifetime is 120 blocks it should be roughly equal 2h), then new transaction probably will be accepted by the node.

## New wallets (> 2.0.4431)

Starting from this version min and max height are set by receiver. If receiver got invitation to transaction it makes kernel's max height as currently known height + lifetime (120 block by default). Sender checks that this height is valid from his known height and sends transaction to the node.

- if sender is in sync and receiver is out-of-sync and its known height is far below sender's (more than lifetime of transaction), then transaction will be failed by sender.

- if sender is out-of-sync, but receiver is in sync and its known height is far above sender's, then transaction will be failed by sender.

- if both sender and receiver are out-of-sync and kernel's maximum height is below than current chain height, then transaction will be failed by the node

- in other cases, kernel with valid range of heights should be created and, as a result, it should most likely be accepted by the node
