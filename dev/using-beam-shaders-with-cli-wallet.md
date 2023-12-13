---
description: >-
  This document describes how to use Beam Shaders with CLI wallet running on a
  local network
---

# Using Beam Shaders with CLI wallet


This page is somewhat obsolete, and is replaced with [Using Beam Shaders](using-beam-shaders.md) tutorial


### Prerequisites

1.  Make sure you have [node running](broken-reference) and connected to local network.

    **Notice:** Shaders are applied after the third fork with height 1500. Therefore, for faster testing, we recommend to define lower heights using the `--Fork1`, `--Fork2`, `--Fork3` attributes.
2. Create at least one Beam node\[link] with `--peer=<ip:port of the first node>` for the transaction replication.
3. Make sure you have Beam [CLI wallet connected to your Beam node](beam-wallets/command-line-wallet.md) or to one of the Beam bootstrap nodes.
4. You must have funds in your wallet to pay the fee.

### Shaders

There are two types of Shaders in Beam:

* **Contract Shader** - implements Smart Contract functionality, is stored on the blockchain and are running on Beam nodes.
* **App Shaders** - implements Smart Contract API and is running in Beam wallet.

_All examples refer to a sample application 'mydapp' which has two_ [_Vault_](https://github.com/BeamMW/beam/tree/master/bvm/Shaders/vault) _Shaders: App Shader(app.wasm) and Contract Shader(contract.wasm)._

### Shader API

First of all, you need to know the available actions of a contract. To see all commands supported by the App Shader enter the following command:

```
./beam-wallet-masternet shader --shader_app_file=mydapp/app.wasm -n <node>
```

\
The output is a json format line:

```
Shader output: {"roles": {"manager": {"create": {},"destroy": {"cid": "ContractID"},"view": {},"view_logs": {"cid": "ContractID"},"view_accounts": {"cid": "ContractID"},"view_account": {"cid": "ContractID","pubKey": "PubKey"}},"my_account": {"view": {"cid": "ContractID"},"get_key": {"cid": "ContractID"},"get_proof": {"cid": "ContractID","aid": "AssetID"},"deposit": {"cid": "ContractID","pkForeign": "PubKey","bCoSigner": "uint32_t","amount": "Amount","aid": "AssetID"},"withdraw": {"cid": "ContractID","pkForeign": "PubKey","bCoSigner": "uint32_t","amount": "Amount","aid": "AssetID"}}}}
```

\
For convenience, let's put it this way:

```json
{
  "roles": {
    "manager": {
      "create": {},
      "destroy": { "cid": "ContractID" },
      "view": {},
      "view_logs": { "cid": "ContractID" },
      "view_accounts": { "cid": "ContractID" },
      "view_account": { "cid": "ContractID", "pubKey": "PubKey" }
    },
    "my_account": {
      "view": { "cid": "ContractID" },
      "get_key": { "cid": "ContractID" },
      "get_proof": { "cid": "ContractID", "aid": "AssetID" },
      "deposit": {
        "cid": "ContractID",
        "pkForeign": "PubKey",
        "bCoSigner": "uint32_t",
        "amount": "Amount",
        "aid": "AssetID"
      },
      "withdraw": {
        "cid": "ContractID",
        "pkForeign": "PubKey",
        "bCoSigner": "uint32_t",
        "amount": "Amount",
        "aid": "AssetID"
      }
    }
  }
}
```

As you can see above, there are two roles in this example: `manager` and `my_account`. Each role has its own actions with or without required attributes. In this example, role `manager` has the following actions:

* `create`
* `destroy`, requires the `cid` attribute
* `view`
* `view_logs`, requires the `cid` attribute
* `view_accounts`, requires the `cid` attribute
* `view_account`, requires `cid` and `pubKey` attributes

_**cid**(i.e. **contract id**), is the frequently required attribute. We get it after our contract has been deployed. The same contract which has been deployed with different attributes, will have different values._

### Commands

All command start with `./beam-wallet-masternet shader`.\
There are required flags that need to be passed to work with the contract:

* `--shader_app_file=<app.wasm>` - for application shader
* `--shader_contract_file=<contract.wasm>` - for contract shader
* `--shader_args="role=<role>,action=<action>"` - shader arguments
* `-n <node>`

**Notice:** _All arguments are passed separated by commas. For example:_

```
--shader_args="role=manager,action=view"
```

_If the action has additional attributes, they also are separated by commas:_

```
--shader_args="role=manager,action=view_logs,cid=d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14"
```

### Deploy contract

To work with the contract, it needs to be deployed. In our Vault example contract, the deployment corresponds to the `manager` role and `create` action. Based on our knowledge, we got the following command to **deploy contract**:

```
./beam-wallet-masternet shader --shader_app_file=mydapp/app.wasm --shader_args="role=manager,action=create" --shader_contract_file=mydapp/contract.wasm -n <node>
```

Output example:

```
Creating new contract invocation tx on behalf of the shader
Contract ID: d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14
    	Comment: create Vault contract	Total fee: 1100000 GROTH
I 2022-06-06.11:43:08.288 [ac09d5dc897647bf876b7d17d8219a77][1] Get proof for kernel: 6625e9f7756a98eb
I 2022-06-06.11:43:08.289 Synchronizing with node: 100% (1/1)
I 2022-06-06.11:43:08.289 Current state is 8-65a2ecdf447ad942
I 2022-06-06.11:43:18.354 Sync up to 9-7d15da24d2717100
I 2022-06-06.11:43:18.354 Synchronizing with node: 0% (0/2)
I 2022-06-06.11:43:18.355 CoinID: Key=mine-1:1:1, Value=8000000000 Maturity=6 Spent, Height=9
I 2022-06-06.11:43:18.356 CoinID: Key=chng-1:0:3958598515398969808, Value=7998900000 Maturity=9 Confirmed, Height=9
I 2022-06-06.11:43:18.356 Synchronizing with node: 50% (1/2)
I 2022-06-06.11:43:18.356 Synchronizing with node: 100% (2/2)
I 2022-06-06.11:43:18.356 Current state is 9-7d15da24d2717100
I 2022-06-06.11:43:18.356 [ac09d5dc897647bf876b7d17d8219a77][1] Get proof for kernel: 6625e9f7756a98eb
I 2022-06-06.11:43:18.357 [ac09d5dc897647bf876b7d17d8219a77] Transaction completed
```

### Command examples

*   View deployed contracts:

    ```
    ./beam-wallet-masternet shader --shader_app_file=mydapp/app.wasm --shader_args="role=manager,action=view" --shader_contract_file=mydapp/contract.wasm -n <node>
    ```

    The output could be like this:

    ```
    Shader output: {"contracts": [{"cid": "d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14","Height": 9}]}
    ```
*   Destroy contract (with `cid` from the example above)

    ```
    ./beam-wallet-masternet shader --shader_app_file=mydapp/app.wasm --shader_contract_file=mydapp/contract.wasm --shader_args="role=manager,action=destroy,cid=d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14" -n localhost:10000
    ```

    Output example:

    ```
    Creating new contract invocation tx on behalf of the shader
    Contract ID: d9c5d1782b2d2b6f733486be480bb0d8bcf34d5fdc63bbac996ed76af541cc14
          Comment: destroy Vault contract	Total fee: 1100000 GROTH
    I 2022-06-06.12:34:15.962 Sync up to 311-788f821396683a25
    I 2022-06-06.12:34:15.962 Synchronizing with node: 0% (0/2)
    I 2022-06-06.12:34:15.967 Synchronizing with node: 0% (0/2)
    I 2022-06-06.12:34:15.967 CoinID: Key=mine-1:1:307, Value=8000000000 Maturity=312 Confirmed, Height=307
    I 2022-06-06.12:34:15.968 CoinID: Key=mine-1:1:308, Value=8000000000 Maturity=313 Confirmed, Height=308
    I 2022-06-06.12:34:15.968 CoinID: Key=mine-1:1:309, Value=8000000000 Maturity=314 Confirmed, Height=309
    I 2022-06-06.12:34:15.968 CoinID: Key=mine-1:1:310, Value=8000000000 Maturity=315 Confirmed, Height=310
    I 2022-06-06.12:34:15.969 CoinID: Key=mine-1:1:311, Value=8000000000 Maturity=316 Confirmed, Height=311
    I 2022-06-06.12:34:15.969 Synchronizing with node: 50% (1/2)
    I 2022-06-06.12:34:15.969 Synchronizing with node: 100% (2/2)
    I 2022-06-06.12:34:15.969 Current state is 311-788f821396683a25
    I 2022-06-06.12:34:15.969 [7b8eb0b0bd2340529996c7598d3ebaff][1] Get proof for kernel: eb45c335bca2e17c
    I 2022-06-06.12:34:15.970 Synchronizing with node: 100% (1/1)
    I 2022-06-06.12:34:15.970 Current state is 311-788f821396683a25
    I 2022-06-06.12:34:26.111 Rolled back to 306-910bc6ca48d05757
    I 2022-06-06.12:34:26.112 Sync up to 312-d156e1bf16939393
    I 2022-06-06.12:34:26.113 Synchronizing with node: 0% (0/2)
    I 2022-06-06.12:34:26.113 CoinID: Key=mine-1:1:307, Value=8000000000 Maturity=312 Confirmed, Height=307
    I 2022-06-06.12:34:26.113 CoinID: Key=mine-1:1:308, Value=8000000000 Maturity=313 Confirmed, Height=308
    I 2022-06-06.12:34:26.114 CoinID: Key=chng-1:0:3958598515398969808, Value=7998900000 Maturity=9 Spent, Height=312
    I 2022-06-06.12:34:26.114 CoinID: Key=chng-1:0:12345322638362229725, Value=7997800000 Maturity=312 Confirmed, Height=312
    I 2022-06-06.12:34:26.115 Synchronizing with node: 50% (1/2)
    I 2022-06-06.12:34:26.115 Synchronizing with node: 100% (2/2)
    I 2022-06-06.12:34:26.115 Current state is 312-d156e1bf16939393
    I 2022-06-06.12:34:26.115 [7b8eb0b0bd2340529996c7598d3ebaff][1] Get proof for kernel: eb45c335bca2e17c
    I 2022-06-06.12:34:26.116 [7b8eb0b0bd2340529996c7598d3ebaff] Transaction completed
    ```
