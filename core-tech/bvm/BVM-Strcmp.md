# Env::Strcmp

```C++
int32_t Strcmp(const char* sz1 , const char* sz2);
```
Equivalent of standard `strcmp`. Compares two null-terminated byte strings lexicographically.

## Parameters
* `sz1` : pointers to the first null-terminated byte string to compare
* `sz2` : pointers to the second null-terminated byte string to compare


## Return value
* Negative value if `sz1` appears before `sz2` in lexicographical order.
* Zero if `sz1` and `sz2` compare equal.
* Positive value if `sz1` appears after `sz2` in lexicographical order.

## Notes
* none

## Example 
