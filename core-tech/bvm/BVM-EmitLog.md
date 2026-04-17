# Env::EmitLog

```C++
uint32_t EmitLog(const void* pKey , uint32_t nKey , const void* pVal , uint32_t nVal , uint8_t nType);
```
Emits new log record denoted by `{nType, [pKey, nKey]}`

## Parameters
* `pKey` : pointer to the key. Key could be any kind of data
* `nKey` : the size of the key
* `pVal` : pointer to the value buffer
* `nVal` : the size of the value buffer
* `nType` : the type of the variable


## Return value
* the previous size of the value of the variable
* 0 if there was no such a variable

## Notes
* Will `Halt()` if either variable key or value is invalid (too long).

## Example 