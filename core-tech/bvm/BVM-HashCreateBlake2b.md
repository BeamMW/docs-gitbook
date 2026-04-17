# Env::HashCreateBlake2b

```C++
HashObj* HashCreateBlake2b(const void* pPersonal , uint32_t nPersonal , uint32_t nResultSize);
```
Allocates the Blake-2B hash processor

## Parameters
* `pPersonal` : pointer to the memory buffer with hash personalization (opaque buffer)
* `nPersonal` : size of the hash personalization buffer
* `nResultSize` : the expected hash result size

## Return value
* hash processor handle (opaque pointer)
* `null` if maximum simultaneous hash objects count is exceeded

## Notes
* none


## Example 