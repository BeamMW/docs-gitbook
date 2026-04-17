# Env::DocGetNum64

```C++
uint8_t DocGetNum64(const char* szID, uint64_t* pOut);
```
Reads application shader input int64 argument with name `szID`

## Parameters
* `szID` : 0-terminated string, the name of the argument
* `pOut` : pointer to the result 

## Return value
* 0 there is no parameter with given name
* 8 otherwise

## Notes
* none

## Example 
```C++

uint64_t myValue = 0;
Env::DocGetNum64("my_value", &myValue);

```
