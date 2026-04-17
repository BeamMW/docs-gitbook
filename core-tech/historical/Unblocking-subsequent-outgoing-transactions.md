> **Historical Proposal** — This document is a draft proposal for unblocking subsequent outgoing transactions via UTXO splitting. It was never implemented. It is preserved for historical reference only.

# Problem description (explained by example)
It's simple: due to atomic UTXO implementation, in case you have 100 Beams in a single UTXO and you want to send 20 Beams to each of the 3 different wallets, the action cannot be completed simultaneously: 
* The 100 Beams UXTO will be locked being an input for the first transaction, with 20 Beams and 79 Beams as outputs (let's assume that Transaction Fee was 1 Beam for simplicity). 
* The 79 Beams UTXO will, in turn, be locked again, splitting into 20 Beams and 59 Beams etc

## Why it's not cool
* Each subsequent transaction needs to wait till the previous transactions will be completed, the waiting time can be as long as 12 hours when the peer doesn't come online
* Currently, the wallet owner can't even create new transaction when UTXO is locked

# Objective (what user wants?)
* Just being able to send any amount up to his maximum balance with the lowest delay time possible
* Similar to other currencies, ordinary user doesn't necessarily needs to be immersed into UTXO complexity 

# What can be possibly done manually by the user
* Option 1: Let user to send to multiple peers in a single transaction. Drawbacks:
  * If one amount was wrong - the entire transaction needs to be restarted - the rest of the peers will wait
  * If one of the peers won't get online - the rest will wait again
* Option 2: Let the user to have a special kind of transaction to split the UTXO he has into UTXO with smaller predefined amounts. Drawbacks:
  * User needs to know about the amounts of his future sends upfront if amounts are predefined
  * The actual transactions will be delayed till the split transaction will succeed

# How can we facilitate user needs in more seamless way?
* Option 1: Have a special "split change" checkbox (set to "on" by default) for every outgoing transaction that will cause the change to arrive in multiple UTXO to ensure the user can always send the amounts as granular as 10% (configurable parameter, 10% can be a default value) of what the user sends on average. Drawback A: might cause to too much fragmentation of user UTXO, consolidation might be required. Drawback B: too many outputs will take larger space in the blockchain block, much higher transaction fee will be expected. Upside: the transactions are always unblocked, seamlessly.
* Option 2: Collect statistics and run some automatic split in the background. Drawbacks and upsides as in the case above. Probably it's always good to have more than a single UTXO (say 50% at most of the current balance if not below 50 Beams).
* Option 3: allow deferred transactions. Assumption: transactions are triggered shortly one after the other (1 minute, configurable parameter). Drawbacks: first transaction might delay others. Upside: less UTXOs are created comparing to the options above, hence less transaction fee should be paid. Point to notice: when UTXO are also received, some of the outgoing transactions can be unblocked immediately. Explained by example:
  * User has 100 Beams and wants to transfer 20, 20 and 20 Beams to different peers
  * User creates the outgoing transaction of 20 Beams. The transaction is not started during 1 minute (configurable parameter). Remark: the transaction could "fire" right away if the user had more than a single UTXO.
  * User creates another transaction of 20 Beams within 1 minute. The previous transaction is still delayed.
  * User creates yet another transaction of 20 Beams within 1 minute. The previous transactions are still delayed.
  * After 1 minute, the first transaction "fires" and the split will be created exactly for all the subsequent transactions.