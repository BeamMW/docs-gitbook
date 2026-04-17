# Env::Memcmp

```C++
int32_t Memcmp(const void* p1 , const void* p2 , uint32_t size);
```
Equivalent of standard `memcmp`. Compares the first `size` characters of given arrays. The comparison is done lexicographically.

## Parameters
* `p1` : pointer to the first memory buffer to compare
* `p2` : pointer to the second memory buffer to compare
* `size`  : number of bytes to examine

## Return value
* Negative value if the first differing byte in `p1` is less than the corresponding byte in `p2`.
* ​0​ if all `size` bytes of `p1` and `p2` are equal.
* Positive value if the first differing byte in `p1` is greater than the corresponding byte in `p2`.

## Notes
* none

## Example 
