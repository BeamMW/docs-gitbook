# Env::DerivePk

```C++
void DerivePk(PubKey& pubKey , const void* pID , uint32_t nID);
```
Generates a public key derived from given `ID`

## Parameters
* `pubKey` : returned public key
* `pID`: pointer to the id buffer
* `nID`: the size of id buffer

## Return value
* none

## Notes
* none

## Example 

```C++

// suppose we have GetContractID() which returns our contract ID 
ContractID cid = GetContractID();
PubKey key;
Env::DerivePk(key, &cid, sizeof(cid));
// at this point key contains public key derived from cid

```