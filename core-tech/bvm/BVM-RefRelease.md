# Env::RefRelease

```C++
uint8_t RefRelease(const ContractID& cid);
```
Releases explicit reference to the contract specified by `cid`

## Parameters
* `cid `  : id of the contract we want to release

## Return value
* always 1

## Notes
* will `Halt()` if there is no reference

## Example 