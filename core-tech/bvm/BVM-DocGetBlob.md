# Env::DocGetBlob

```C++
uint32_t DocGetBlob(const char* szID, void* pOut, uint32_t nLen);
```
Reads application shader input binary hex argument with name `szID`

## Parameters
* `szID` : 0-terminated string, the name of the argument
* `pOut` : pointer to the result buffer
* `nLen` : the size of the result buffer

## Return value
* 0 if failed
* size of blob in bytes otherwise

## Notes
* if blob length is bigger than the requested size `nLen` - return the blob size
* if parsed less than requested - return size parsed.
* the requested size is returned only if it equals to the blob size, and was successfully parsed

## Example 
```C++

static const char szName[] = "contract.shader";
// request size of the buffer
uint32_t nSize = Env::DocGetBlob(szName, nullptr, 0);
if (!nSize)
    return 0;

// allocate buffer
void* p = Env::StackAlloc(nSize);
Env::DocGetBlob(szName, p, nSize);

```
