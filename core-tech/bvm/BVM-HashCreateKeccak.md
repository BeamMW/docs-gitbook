# Env::HashCreateKeccak

```C++
HashObj* HashCreateKeccak(uint32_t nBits);
```
Allocates the Keccak hash processor

## Parameters
* `nBits` : the length of the hash in bits

## Return value
* hash processor handle (opaque pointer)
* `null` if maximum simultaneous hash objects count is exceeded

## Notes
* none


## Example 