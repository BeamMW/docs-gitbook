# Env::DocAddText

```C++
void DocAddText(const char* szID, const char* val);
```
Emits text JSON field with name `szID` and value `val`

## Parameters
* `szID` : 0-terminated string, the name of the field
* `val` : 0-terminated string, the value

## Return value
* none

## Notes
* none

## Example 
```C++
DocAddText("my_field", "abra");
```
result
```json
 "my_field" : "abra"
```