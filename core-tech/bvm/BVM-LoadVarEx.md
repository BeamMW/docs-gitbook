# Env::LoadVarEx

```C++
void LoadVarEx(void* pKey,
               uint32_t& nKey,
               uint32_t nKeyBufSize,
               void* pVal,
               uint32_t& nVal,
               uint8_t nType,
               uint8_t nSearchFlag)
```
Loads the value of the contract variable denoted by `{nType, [pKey, nKey]}` into buffer pointed to by `pVal`.

## Parameters
* `pKey` : pointer to the key. Key could be any kind of data
* `nKey` : the size of the key
* `nKeyBufSize` : 
* `pVal` : pointer to the value buffer
* `nVal` : the size of the buffer
* `nType` : can be anything. Means - the contract can read the auxiliary variables (such as total locked funds) that BVM uses for it.
* `nSearchFlag` :


## Return value
* the actual (non-truncated) value size or 0 if such a variable doesn't exist.

## Notes
* If the value is larger than `nVal` - it's truncated.

## Example 