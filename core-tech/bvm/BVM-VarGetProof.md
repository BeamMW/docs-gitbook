# Env::VarGetProof

```C++
uint32_t VarGetProof(const void* pKey , uint32_t nKey ,
                     const void** ppVal , uint32_t* pnVal , const Merkle::Node** ppProof);
```
Gets Merkle proof for the variable with given key

## Parameters
* `pKey` : pointer to the key buffer
* `nKey` : the size of the key buffer
* `ppVal`: pointer to the address of the value buffer
* `pnVal`: pointer to the address of the value buffer size
* `ppProof`: pointer to the address of the Merkle proof buffer

## Return value
* the number of nodes in the Merkle proof

## Notes
* none

## Example 