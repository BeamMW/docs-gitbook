# Env::AssetDestroy

```C++
uint8_t AssetDestroy(AssetID aid);
```
Destroys the asset type specified by `aid`.

## Parameters
* `aid`  : asset id

## Return value
* 1 if successful
* 0 otherwise

## Notes
* `Halt()` if asset specified by `aid` was not created by this contract
* fails if the asset can't be destroyed (not fully burned, lock time didn't elapse, etc.)

## Example 