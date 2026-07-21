# Env::FundsUnlock

```C++
void FundsUnlock(AssetID aid , Amount amount);
```
Unlocks the specified `amount` of asset specified by `aid` to the current transaction

## Parameters
* `aid `  : asset id 
* `amount`: amount to unlock

## Return value
* none

## Notes
* will `Halt()` if the contract doesn't currently have sufficient `amount` of this asset locked.

## Example 