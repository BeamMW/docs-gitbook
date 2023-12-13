---
description: Documentation for the Beam DEX contract and application
---

# Beam DEX

### Details:



Repository: [https://github.com/BeamMW/beam/tree/master/bvm/Shaders/amm](https://github.com/BeamMW/beam/tree/master/bvm/Shaders/amm)

Dappnet CID: 4e0a28b2b2a83b811ad17ba8228b0645dbce2969fd453a68fbc0b60bc8860e02



Status: In development (14.11.2022)

### Shader API



To learn how to use Shader API pleasse refer to [using-beam-shaders-with-cli-wallet.md](using-beam-shaders-with-cli-wallet.md "mention")



Some notes regarding the API:



* pool\_view and pools\_view methods can be called by user role
* "kind": "uint32\_t" parameter has three values for different types of pool with different fees:\
      0 : low volatility, fee is 0.05%\
      1 :  mid volatility, fee is 0.3%\
      2 : high volatility, fee is 1% (default value)\




```json
{
    "roles":
    {
        "admin":
        {
            "view":
            {},
            "destroy":
            {
                "cid": "ContractID"
            },
            "deploy":
            {
                "hUpgradeDelay": "Height",
                "nMinApprovers": "uint32_t",
                "bSkipVerifyVer": "uint32_t",
                "cidDaoVault": "ContractID"
            },
            "schedule_upgrade":
            {
                "cid": "ContractID",
                "hTarget": "Height",
                "bSkipVerifyVer": "uint32_t",
                "iSender": "uint32_t",
                "approve_mask": "uint32_t"
            },
            "replace_admin":
            {
                "cid": "ContractID",
                "iAdmin": "uint32_t",
                "pk": "PubKey",
                "iSender": "uint32_t",
                "approve_mask": "uint32_t"
            },
            "set_min_approvers":
            {
                "cid": "ContractID",
                "newVal": "uint32_t",
                "iSender": "uint32_t",
                "approve_mask": "uint32_t"
            },
            "explicit_upgrade":
            {
                "cid": "ContractID"
            },
            "pool_view":
            {
                "cid": "ContractID",
                "aid1": "AssetID",
                "aid2": "AssetID",
                "kind": "uint32_t"
            },
            "pools_view":
            {
                "cid": "ContractID"
            }
        },
        "user":
        {
            "pool_create":
            {
                "cid": "ContractID",
                "aid1": "AssetID",
                "aid2": "AssetID",
                "kind": "uint32_t"
            },
            "pool_destroy":
            {
                "cid": "ContractID",
                "aid1": "AssetID",
                "aid2": "AssetID",
                "kind": "uint32_t"
            },
            "add_liquidity":
            {
                "cid": "ContractID",
                "aid1": "AssetID",
                "aid2": "AssetID",
                "kind": "uint32_t",
                "val1": "Amount",
                "val2": "Amount",
                "bPredictOnly": "uint32_t"
            },
            "withdraw":
            {
                "cid": "ContractID",
                "aid1": "AssetID",
                "aid2": "AssetID",
                "kind": "uint32_t",
                "ctl": "Amount",
                "bPredictOnly": "uint32_t"
            },
            "trade":
            {
                "cid": "ContractID",
                "aid1": "AssetID",
                "aid2": "AssetID",
                "kind": "uint32_t",
                "val1_buy": "Amount",
                "bPredictOnly": "uint32_t"
            }
        }
    }
}
```

