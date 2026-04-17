# Env::Comm_Listen

```C++
void Comm_Listen(const void* pID, uint32_t nID, uint32_t nCookie);
```
Loads the value of the contract variable denoted by `{nType, [pKey, nKey]}` into buffer pointed to by `pVal`.

## Parameters
* `pID` : pointer to the key. Key could be any kind of data
* `nID` : the size of the key
* `nCookie` : pointer to the value buffer


## Return value
* none

## Notes
* none

## Example 