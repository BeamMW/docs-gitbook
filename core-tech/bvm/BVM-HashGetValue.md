# Env::HashGetValue

```C++
void HashGetValue(HashObj* pHash , void* pDst , uint32_t size);
```
Read the resulting hash value calculated by the hash processor referenced by the handle `pHash`

## Parameters
* `pHash` : handle to the hash processor
* `pDst` : pointer to the memory buffer to fill the hash
* `size` : size of the memory buffer

## Return value
* none

## Notes
* If the actual hash result is longer than `size` - it's truncated
* If the actual hash result is shorter than `size` - the remaining part of the result is zeroed
* The hash object is left 'intact'. Means - it's possible to add more data and get updated hash result.


## Example 