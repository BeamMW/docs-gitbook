# Env::VerifyBeamHashIII

```C++
uint8_t VerifyBeamHashIII(const void* pInp , uint32_t nInp , const void* pNonce , 
                          uint32_t nNonce , const void* pSol , uint32_t nSol);
```
Verifies the BeamHashIII PoW solution for the specified input and nonce

## Parameters
* `pInp` : pointer to the input memory buffer (hash value)
* `nInp` : the size of the input buffer
* `pNonce`: pointer to the nonce buffer
* `nNonce`: the size of the nonce buffer
* `pSol`: pointer to the buffer in memory, containing the solution of BeamHashIII PoW algorithm
* `nSol`: the size of the solution buffer

## Return value
* 1 if the solution is valid
* 0 otherwise

## Notes
* none

## Example 