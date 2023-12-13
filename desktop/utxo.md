---
description: >-
  Since cryptocurrency is, by definition, digital, UTXO's function the same way
  dollar bills do with fiat currency. If your physical wallet carries banknotes,
  then your digital wallet carries UTXOs.
---

# UTXO


The Beam wallet minimizes the output value you receive by automatically selecting the UTXO for each transaction. You will not be able to send or spend your UTXOs until the transaction is complete.


## UTXO in Your Beam Wallet

An unspent transaction output (UTXO) represents the amount of cryptocurrency left after each transaction which then records the input for new transactions. The remaining outputs in your Beam wallet can be found in the **UTXO**<img src=".gitbook/assets/Screen Shot 2021-07-03 at 5.07.08 PM.png" alt="" data-size="line">tab.

![](<.gitbook/assets/Screen Shot 2021-05-28 at 6.33.10 PM.png>)



## Type of UTXO

| Type                | Description                                                                                                                                                                                                                                                                         |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Regular**         | UTXO received as a result of a transaction. It is immediately available for spending                                                                                                                                                                                                |
| **Change**          | UTXO received as a result of a change from a transaction. It is immediately available for spending                                                                                                                                                                                  |
| **Transaction fee** | Fees received as a result of mining a block that contains transactions                                                                                                                                                                                                              |
| **Coinbase**        | UTXO you have mined. It has the maturity of 3 hours (240 blocks) and will not be immediately seen in the Available tab                                                                                                                                                              |
| **Shielded**        | UTXO received as a result of an Offline or Max privacy transaction. Offline is immediately available for spending, meanwhile received max privacy coins can be made immediately available (no limit) or locked by the “Max privacy longest transaction time” parameter in settings. |



