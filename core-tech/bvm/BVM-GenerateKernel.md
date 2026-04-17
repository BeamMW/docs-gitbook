# Env::GenerateKernel

```C++
void GenerateKernel(const ContractID* pCid , uint32_t iMethod ,
                    const void* pArg , uint32_t nArg , const FundsChange* pFunds , uint32_t nFunds ,
                    const SigRequest* pSig , uint32_t nSig , const char* szComment , uint32_t nCharge);
```
Generates kernel for which makes node to call method `iMethod` of contract with `pCid`

## Parameters
* `pCid` : pointer to contract ID
* `iMethod ` : method number to call
* `pArg` : pointer to arguments buffer
* `nArg` : length of the arguments buffer
* `pFunds` : optional pointer to an array of [FundsChange](FundsChange) structures
* `nFunds` : number of elements in the above array
* `pSig` : optional pointer to an array of [SigRequest](SigRequest) structures
* `nSig` : number of elements in the above array
* `szComment` : Transaction comment (0-terminated character string)
* `nCharge` : estimate of the BVM charge (in charge uints) that the transaction execution is supposed to consume

## Return value
* none

## Notes

App shader calls this function to request the wallet to create the transaction that is supposed to either deploy, destroy, or invoke a contract method on a blockchain.

Set `iMethod` to `0` if the contract is being-deployed, in this case `pCid` (pointer to `ContractID`) is ignored. Otherwise the `pCid` must point to a valid `ContractID`.

The `pArgs`/`nArgs` describe the proprietary structure with the parameters as expected by the shader's method.

The `pFunds`/`nFunds` is the array description of `FundsChange` structures, which describe the funds that should be consumed/received in the transaction. The app shader should specify such a structure for each asset type. The wallet selects transaction inputs/outputs appropriately.
See description of `FundsChange` for more info.

The `pSig`/`nSig` is the array description of `SigRequest` structures, which should correspond to the keys by which the kernel must be signed. App shader should specify such a structure for each key requested by the contract shader via `AddSig`, <u>in the same order</u>.

`nCharge` is a rough estimate of how much BVM charge the contract invocation would consume. Based on it the wallet calculates the minimum transaction fee.
For "simple" methods, that don't perform excessive computations or I/O app shader may specify `0`. The default minimum fee should be enough.

## Example 

