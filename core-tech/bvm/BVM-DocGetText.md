# Env::DocGetText

```C++
uint32_t DocGetText(const char* szID, char* szRes, uint32_t nLen);
```
Reads application shader input text argument with name `szID`. Text could be a unquoted string of characters, in this case ' ' or ',' are not allowed, or text could be a quoted string, in this case any printable characters are allowed, but '"' should be escaped with '\\'. For example:

`data=some_text_without_spaces0x134`

`args="Pretty text \"some name\", a = \"test\" n=145 "`

## Parameters
* `szID` : 0-terminated string, the name of the argument
* `szRes` : pointer to the result buffer
* `nLen` : the size of the result buffer

## Return value
* the size of the result in bytes

## Notes
* the function copies only `min(nLen, <result_size + 1>)` bytes
* the result is always 0-terminated

## Example 
```C++

char szAction[0x20];
Env::DocGetText("action", szAction, sizeof(szAction));

```
