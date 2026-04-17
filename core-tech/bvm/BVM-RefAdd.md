# Env::RefAdd

```C++
uint8_t RefAdd(const ContractID& cid);
```
Adds explicit reference to the contract specified by `cid`.
 
Can be called many times, the reference counter is increased appropriately

## Parameters
* `cid` : identifier of the contract we want to add a reference

## Return value
* 1 if successful, 0 if the contract doesn't exist


## Example