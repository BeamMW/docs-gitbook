# Env::Strlen

```C++
uint32_t Strlen(const char* sz);
```
Equivalent of standard `strlen`. Returns the length of the given byte string, that is, the number of characters in a character array whose first element is pointed to by `sz` up to and not including the first null character.

## Parameters
* `sz` : pointer to the null-terminated byte string to be examined

## Return value
* The length of the null-terminated string `sz`.

## Notes
* none

## Example 
