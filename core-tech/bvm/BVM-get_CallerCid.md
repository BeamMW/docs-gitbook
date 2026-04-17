# Env::get_CallerCid

```C++
void get_CallerCid(uint32_t iCaller , ContractID& cid);
```
Gets `cid` to the callee contract ID at the depth specified by `iCaller`

## Parameters
* `iCaller `  : the depth in the call stack
* `cid` : reference to store result value

## Return value
* `cid` is filled with caller id if succeeded

## Notes
* set `iCaller == 0` to get the current contract ID
* will Halt() if `iCaller` is invalid

## Example 