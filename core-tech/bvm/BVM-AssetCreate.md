# Env::AssetCreate

```C++
AssetID AssetCreate(const void* pMeta , uint32_t nMeta);
```
Creates a new asset type withe metadata specified by `[pMeta, nMeta]`

## Parameters
* `pMeta`  : pointer to the buffer with metadata
* `nMeta`  : the size of the metadata buffer

## Return value
* asset ID if successful
* 0 otherwise.

## Notes
* fails in case of duplication (if this contract already created asset with this exact metadata)
* this operation is not free, you have to make a deposit of 10 Beam. This amount should be available in your wallet. Also you have to fill [FundsChange](FundsChange) structure

## Example 