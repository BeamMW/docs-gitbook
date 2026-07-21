# Env::Vars_Enum

```C++
uint32_t Vars_Enum(const void* pKey0 , uint32_t nKey0 , const void* pKey1 , uint32_t nKey1);
```
Starts variable enumeration process in key range `[key0, key1]`

## Parameters
* `pKey0`  : pointer to the lower bound key buffer
* `nKey0`  : the size of the lower bound key buffer
* `pKey1`  : pointer to the upper bound key buffer
* `nKey1`  : the size of the upper bound key buffer

## Return value
* the slot number which could be used to work with this enumeration

## Notes
* the slot number should be used with [Vars_MoveNext](BVM-Vars_MoveNext.md) and [Vars_Close](BVM-Vars_Close.md) functions

## Example 