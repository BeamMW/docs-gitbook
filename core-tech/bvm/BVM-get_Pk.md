# Env::get_Pk

```C++
void get_Pk(Secp_point& res, const void* pID, uint32_t nID);
```
Generates secret scalar using given `ID` data, and multipies it to the default generator point `G`

## Parameters
* `res` : result point
* `pID` : pointer to the buffer with `id` data
* `nID` : size of the buffer


## Return value
* none

## Notes
* none

## Example 