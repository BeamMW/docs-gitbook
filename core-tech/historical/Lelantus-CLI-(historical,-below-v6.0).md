> **Historical Document** — This document covers Lelantus CLI usage for wallet versions below v6.0. It is retained for historical reference only. For v6.0 and later, see [Transactions-Lelantus-Shielded-Pool](../transactions/Transactions-Lelantus-Shielded-Pool.md).

## Overview
Conceptually, Lelantus is a mean which allows to avoid UXTO linkability in transactions graph. To make UTXOs unlinked user should insert regular BEAM UTXO into _shielded pool_, converting these UTXOs into _shielded UTXOs_ and then, after some time extract them back as _unlinked_ UTXOs. _Shielded UTXO_ belonging to the wallet could be detected by the node with owner key (as regular utxo), and the user can use this info to extract these coins back.
We are trying to hide the details of this process from the user, extracting from the _shielded pool_ is performed automatically when user sends beams, it influences the fee, but the user shouldn't think about unlinking.

To send BEAMs through Lelantus we introduced three types of addresses:

* offline
* max privacy
* public offline

All of these addresses could be used with `send` command and should be passed via `-r` parameter

###  Offline address
It allows sending BEAMs without interactions with the receiver. It is a very long base58 string, which contains embedded information for 10 payments(in UI), also it has SBBS address of the receiver, so, in the case when the receiver becomes online, the sender could get info for more payments.

To generate this type of address with 10 embedded payments use
```
./beam-wallet get_address --offline=10
```

### Max privacy address
This address allows to make only one payment, there is no ability to get more payments, and there is a guaranty that shielded coins received by sending on this address will be extracted in the correct moment to achieve the max privacy effect. 
To generate a new max privacy address use
```
./beam-wallet get_address --max_privacy 
```

### Public offline address
This type of address is intended to be used to receive donations. It is permanent, relatively short, and provides less privacy comparing with others, the sender has an ability to detect if the sent shielded coin has been spent.
This type of address could be obtained with the following command:
```
./beam-wallet get_address --public_offline 
```

## Offline payments
With Lelantus we are able to without need to be online, this is also known as one-side payments. To make them we need to accomplish the following steps:

### Receiver
1. Receiver have to generate a token with the needed quantity of vouchers (allowed number of payments)
    ```
    ./beam-wallet get_address --offline=3 
    ```

1. Send this token to the sender, and go offline 

### Sender 
1. Sender can use the received token to make payments as many times as many vouchers this token has.
    ```
    ./beam_wallet insert_to_pool -r <token> -n <node address> -a <amount> -f <fee>
    ```

### Receiver
1. Receiver after a while should check if he has _shielded UTXOs_
    ```
    ./beam_wallet info
    ```
    the shielded coins has type `shld`
    ```
    |                                                ID |           BEAM |          GROTH | Maturity           | Status               | Type     |
                                                   14724               34                0   95849                [Spent]                shld
                                                   14725                7                0   95873                [Spent]                shld
    ```
1. Also a new transaction record should appear with status `received max privacy`, `received offline` or `received public offline`, depending od what address type was used 

    ```
    ./beam_wallet info --tx_history
    ```
    ```
    2020.11.14 12:52:22    incoming                                     5   received offline        c714875164c8444cb12f90c4353fa1f6
    2020.10.30 18:13:22    incoming                                     1   received max privacy    7a6dbf11fa6649e39992c41c349ab6ad
    2020.10.30 17:57:22    incoming                                     4   received max privacy    d5f1b5acee474b4cb187321431040306
    2020.10.30 11:15:22    incoming                                     2   received public offline
    ```