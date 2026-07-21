# Env::DocAddArray

```C++
void DocAddArray(const char* szID);
```
Emits beginning of array with name `szID`

## Parameters
* `szID` : 0-terminated string, the name of the field

## Return value
* none

## Notes
* none

## Example 
```C++
DocAddArray("my_array");
```
result
```json
 "my_array" : [
```