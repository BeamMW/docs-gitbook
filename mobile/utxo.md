---
description: >-
  Since cryptocurrency is, by definition, digital, UTXO's function the same way
  dollar bills do with fiat currency. If your physical wallet carries banknotes,
  then your digital wallet carries UTXOs.
---

# UTXO




**What is a UTXO, and how does it apply to your Beam coins?**

1. **UXTO (unspent transaction value)**.\
   UTXO represents the amount of cryptocurrency remaining after each transaction. The "output" for each UXTO creates a new "input" for new transactions.
2.  **Your Beam wallet automatically selects the UTXO for each transaction**.

    Your Beam wallet automatically minimizes the output value you receive by automatically selecting the UTXO for each transaction.

    You will not be able to send or spend your UTXOs until each transaction is complete.                                                                           


## UTXO screen

Access your wallet UTXO's by tapping **menu**<img src=".gitbook/assets/Screen Shot 2021-07-01 at 9.19.58 PM (1).png" alt="" data-size="line">**> UTXO**<img src=".gitbook/assets/IMG_3379.jpg" alt="" data-size="line">in your Beam mobile wallet.

![](.gitbook/assets/photo\_2021-05-27\_19-02-14.jpg)

## Types of UTXO

| Type                | Description                                                                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Regular**         | UTXO received as a result of a transaction. It is immediately available for spending                                                                                                                                                                                            |
| **Change**          | UTXO received as a result of change leftover from a transaction. It is immediately available for spending.                                                                                                                                                                      |
| **Transaction fee** | Fees received as a result of mining a block which that contains transactions.                                                                                                                                                                                                   |
| **Coinbase**        | UTXO you have mined. It has a maturity of 3 hours (240 blocks) and will not be immediately seen in your wallet.                                                                                                                                                                 |
| **Shielded**        | UTXO received as a result of an Offline or Max privacy transaction. Offline is immediately available for spending, meanwhile received Max Privacy coins can be made immediately available (no limit) or locked by “Max Privacy longest transaction time” parameter in settings. |

