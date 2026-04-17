# Env::DocAddGroup

```C++
void DocAddGroup(const char* szID);
```
Generates begining of JSON group with name `szID`

## Parameters
* `szID` : 0-terminated string, the name of group

## Return value
* none

## Notes
* none

## Example 
```C++
DocAddGroup("new_group");
```
result
```json
 "new_group": {
```
