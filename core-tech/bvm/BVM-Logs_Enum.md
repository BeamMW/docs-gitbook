# Env::Logs_Enum

```C++
uint32_t Logs_Enum(const void* pKey0 , uint32_t nKey0 , const void* pKey1 , uint32_t nKey1 , const HeightPos* pPosMin , const HeightPos* pPosMax);
```
Starts log enumeration process in key range `[key0, key1]`

## Parameters
* `pKey0`  : pointer to the lower bound key buffer
* `nKey0`  : the size of the lower bound key buffer
* `pKey1`  : pointer to the upper bound key buffer
* `nKey1`  : the size of the upper bound key buffer
* `pPosMin`:
* `pPosMax`:

## Return value
* the slot number which could be used to work with this enumeration

## Notes
* the slot number should be used with [Logs_MoveNext](BVM-Logs_MoveNext.md) and [Logs_Close](BVM-Logs_Close.md) functions

## Example 