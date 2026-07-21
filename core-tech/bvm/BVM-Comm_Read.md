# Env::Comm_Read

```C++
uint32_t Comm_Read(void* pBuf, uint32_t nSize, uint32_t* pCookie, uint8_t bKeep);
```
Loads the value of the contract variable denoted by `{nType, [pKey, nKey]}` into buffer pointed to by `pVal`.

## Parameters
* `pKey` : pointer to the key. Key could be any kind of data
* `nKey` : the size of the key
* `pVal` : pointer to the value buffer
* `nVal` : the size of the buffer
* `nType` : can be anything. Means - the contract can read the auxiliary variables (such as total locked funds) that BVM uses for it.


## Return value
* the actual (non-truncated) value size or 0 if such a variable doesn't exist.

## Notes
* none

## Example 