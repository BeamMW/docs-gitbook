# Env::DocAddNum64

```C++
void DocAddNum64(const char* szID, uint64_t val);
```
Emits 64-bit integer JSON field with name `szID` and value `val`

## Parameters
* `szID` : 0-terminated string, the name of the field
* `val` : the value

## Return value
* none

## Notes
* none

## Example 
```C++
DocAddNum64("int_field", 42);
```
result
```json
 "int_field" : 42
```