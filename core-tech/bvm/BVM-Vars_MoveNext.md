# Env::Vars_MoveNext

```C++
uint8_t Vars_MoveNext(uint32_t iSlot , void* pKey , uint32_t& nKey , void* pVal , uint32_t& nVal , uint8_t nRepeat);
```
Reads the next variable specified by `iSlot`

## Parameters
* `iSlot` : enumeration slot returned by [Vars_Enum](Vars_Enum)
* `pKey` : pointer to the key buffer
* `nKey` : the size of the key buffer
* `pVal` : pointer to the value buffer
* `nVal` : the size of the value buffer
* `nRepeat` : 1 - don't move read current variable again, 0 - read next variable

## Return value
* 1 if successful
* 0 otherwise

## Notes
* the slot number should be obtained with [Vars_Enum](Vars_Enum) function
* this function copies `nKey` bytes to `pKey` and then overrides `nKey` with exact size of the key
* this function copies `nVal` bytes to `pVal` and then overrides `nVal` with exact size of the value

## Example 