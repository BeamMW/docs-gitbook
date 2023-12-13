# Beam Fierce Fermion 6.0 Upgrade Guide for pools and exchanges

Beam Fierce Fermion 6.0 includes breaking Hard Fork that will be activated on height 1,280,000 (around 15:30 UTC, 13th of June 2021). It is important to complete all upgrade before that date.

## What new in version 6.0 
1. Smart contracts.
1. Confidential Assets support in wallet API. Read more [here](https://github.com/BeamMW/beam/wiki/Adding-support-for-Beam-Confidential-Asset)
1. New types of transaction are available in the wallet API:
    * Maximum privacy transaction
    * Offline transaction

    In order to receive these transactions, there are no need to make any changes in code, but your wallet should be connected to the node with `owner_key`. If you want to send them you have to explicitly enable them in the wallet, and distinguish new address types.
1. New address types in wallet API. We highly recommend to use these new addresses, this should bring better user experience both for sending and receiving. For backward compatibility, we preserved old address type.
The difference between new addresses and old are the following
    * new addresses are `base58` encoded and have relatively long, old addresses are `hex` encoded and have length 64-67 characters
    * new addresses can hold additional data for transaction, this allows us to make new type of transactions mentioned before and add more when it will be needed
1. Starting from 6.0 Beam address should be validated using the following regular expression:

    ```
    /[0-9a-zA-Z]{64,500}/
    ```
1. Minimal fee has changed
    * For the _regular_ transaction has been increased up to 100,000 Groth
    * For _offline_ and _max privacy_ transaction it became 1,100,000 Groth (+1,000,000 Groth for each shielded output)


## Upgrading the node and the wallet

> IMPORTANT

 Before starting the upgrade procedure it is highly recommended to stop the wallets and back up the _wallet.db_ and _node.db_ files.

1. Download the new binaries from [Beam Website](/downloads) or build from [source](https://github.com/BeamMW/beam/tree/mainnet)
1. [Enable](https://github.com/BeamMW/beam/wiki/Adding-support-for-Beam-Confidential-Asset) CA support if needed
1. Update address verification. Now, `validate_address` method returns additional info about address:

    `<--`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 1,
        "result" : 
        {
            "is_valid" : true,
            "is_mine" : false,
            "type": "offline",  // <---------- NEW
            "payments": 3       // <---------- NEW
        }
    }
    ```

    * `type` the type of address, possible values are
        * `regular` - regular old-style `hex` encoded address, default
      * `offline` - new-style `base58` encoded address for _offline_ transaction, could be used several times, it has no strong guaranties for privacy
      * `max_privacy` - new-style `base58` encoded address for _maximum privacy_ transaction, could be used once
      * `public_offline` - new-style `base58` encoded address for _offline_ transaction, it's constant, it provided less privacy, you could publish it, designed for donations
      * `regular_new` - new-style `base58` encoded address for regular mimblewimble transaction

    * `payments` - the number of payments left in case of `offline` address

    If you don't want to support newer type of transactions you could simply allow `regular` addresses.

    NOTE: `offline` address has all the data to make _regular_ online transaction using MimbleWimble protocol. You can make both type of transaction using this type of address. If you want to send regular online transaction you should pass `offline: false`
    ```json
    {
        "jsonrpc":"2.0", 
        "id": 2,
        "method":"tx_send", 
        "params":
        {
            "value": 12342342,
            "fee": 2,
            "from": "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67",
            "address": "2DWjfstQoaLMYYgvc4NBTTGqRdyM3nvWxP9bDHN....",  // <-- offline address 
            "comment": "thank you!",
            "asset_id": 1,
            "offline": false   // <-- make regular online payment
        }
    }

1. regex for the newer address is:
    ```
    /[0-9a-zA-Z]{64,500}/
    ```
    it allows both old-style and new-style addresses.
1. in order to generate new-style address pass `"type"` parameter to `create_address` method

    `-->`
    ```json
    {
        "jsonrpc" : "2.0", 
        "id" : 1,
        "method" : "create_address", 
        "params" :
        {
            "type": "max_privacy",  // <----- NEW
            "expiration" : "auto",
            "comment" : "John Smith"
        }
    }
    ```
    The values for `"type"` parameter are the same as described earlier. If you don't specify `"type"` `"regular"` is used by default to keep backward compatibility.
1. If you want to send to _offline_ and _max privacy_ addresses run API with `--enable_lelantus`.

    ```
    ./wallet-api --enable_lelantus -n <node address>
    ```
   or specify it in config file
   ```
   enable_lelantus=true
   ```







   
 