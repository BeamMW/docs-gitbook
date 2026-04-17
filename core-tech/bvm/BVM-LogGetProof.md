# Env::LogGetProof

```C++
uint32_t LogGetProof(const HeightPos& pos , const Merkle::Node** ppProof);
```
Gets Merkle proof for the log at given `pos`

## Parameters
* `pos` : pointer to the key buffer
* `ppProof`: pointer to the address of the Merkle proof buffer

## Return value
* the number of nodes in the Merkle proof

## Notes
* none

## Example 