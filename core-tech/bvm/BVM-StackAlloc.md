# Env::StackAlloc

```C++
void* StackAlloc(uint32_t size);
```
Equivalent of standard `alloca`. Allocates the specified `size` on the stack, and returns the pointer to allocated region.

## Parameters
* `size` : number of bytes to allocate

## Return value
* pointer to allocated region

## Notes
* `Halt()` if can't allocate.

## Example 
