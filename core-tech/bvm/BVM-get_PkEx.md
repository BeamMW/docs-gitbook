# Env::get_PkEx

```C++
void get_PkEx(Secp_point& res, const Secp_point& gen, const void* pID, uint32_t nID)
```
Generates secret scalar using given `ID` data, and multipies it to the given generator point

## Parameters
* `res` : result point
* `gen` : custom generator point
* `pID` : pointer to the buffer with `id` data
* `nID` : size of the buffer


## Return value
* none

## Notes
* none

## Example 