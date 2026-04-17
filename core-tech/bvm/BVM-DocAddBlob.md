# Env::DocAddBlob

```C++
void DocAddBlob(const char* szID, const void* pBlob, uint32_t nBlob);
```
Emits JSON blob field with name `szID`, the blob is binary data, the field is hex-encoded string

## Parameters
* `szID` : 0-terminated string, the name of the field
* `pBlob` : pointer to the blob
* `nBlob` : the size of blob

## Return value
* none

## Notes
* none

## Example 
```C++

ContractID cid = GetContractID();
DocAddBlob("my_cid", &cid, sizeof(cid));

```
result
```json
 "my_cid" : "42dabc5....ee00s7"
```