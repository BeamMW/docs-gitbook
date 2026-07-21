# Env::AssetEmit

```C++
uint8_t AssetEmit(AssetID aid , Amount amount , uint8_t bEmit);
```
Emits or burns the specified `amount` of the specified asset type

## Parameters
* `aid`  : asset id
* `amount`: the amount to emit or burn
* `bEmit` : flag, 0 - burn amount of the asset, 1 - emit

## Return value
* 1 if successful
* 0 otherwise

## Notes
* `Halt()` if asset specified by `aid` was not created by this contract
* fails in case of overflow (i.e. attempt to burn more than was emitted)
* the emitted/burned asset is NOT automatically added/subtracted to/from the current transaction. It's only locked/unlocked to the current contract
* to move it to the current transaction call [FundsLock](BVM-FundsLock.md) / [FundsUnlock](BVM-FundsUnlock.md) explicitly.

## Example 