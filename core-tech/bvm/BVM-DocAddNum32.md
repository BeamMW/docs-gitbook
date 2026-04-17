# Env::DocAddNum32

```C++
void DocAddNum32(const char* szID, uint32_t val);
```
Emits 32-bit integer JSON field with name `szID` and value `val`

## Parameters
* `szID` : 0-terminated string, the name of the field
* `val` : the value

## Return value
* none

## Notes
* none

## Example 
```C++
DocAddNum32("int_field", 42);
```
result
```json
 "int_field" : 42
```