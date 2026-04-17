# Env::SaveVar

```C++
uint32_t SaveVar(const void* pKey , uint32_t nKey , const void* pVal , uint32_t nVal , uint8_t nType);
```
Save the new value of the variable denoted by `{nType, [pKey, nKey]}`

## Parameters
* `pKey` : pointer to the key. Key could be any kind of data
* `nKey` : the size of the key
* `pVal` : pointer to the value buffer
* `nVal` : the size of the value buffer
* `nType` : the type of the variable (see [KeyTag](KeyTag) for possible values)


## Return value
* the previous size of the value of the variable
* 0 if there was no such a variable

## Notes
* if `nVal` is zero then the variable is deleted
* Will `Halt()` if either variable key or value is invalid (too long).
* Set `nType` to `KeyTag::Internal` for normal variables, or `KeyTag::InternalStealth` for 'stealth' (those are not stored in MMR), other variable types are read-only.
## Example 