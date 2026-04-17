# Env::DocGetNum32

```C++
uint8_t DocGetNum32(const char* szID, uint32_t* pOut);
```
Reads application shader input int32 argument with name `szID`

## Parameters
* `szID` : 0-terminated string, the name of the argument
* `pOut` : pointer to the result 

## Return value
* 0 there is no parameter with given name
* 4 otherwise

## Notes
* none

## Example 
```C++

uint32_t myValue = 0;
Env::DocGetNum32("my_value", &myValue);

```
