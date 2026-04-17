# Env::SlotInit

```C++
void SlotInit(const void* pExtraSeed, uint32_t nExtraSeed, uint32_t iSlot)
```
Generates a new random nonce value and places it into given slot. Extra seed value could be passes to random generator.

## Parameters
* `pExtraSeed` : pointer to the extra seed value buffer
* `nExtraSeed` : size of extra seed value
* `iSlot` : pointer to the value buffer

## Return value
* none

## Notes
* none

## Example 