# Env::Heap_Free

```C++
void Heap_Free(void* pPtr);
```
Frees the memory allocated previously on heap with [Heap_Alloc](BVM-Heap_Alloc.md)

## Parameters
* `pPtr` : pointer to the memory region to free

## Return value
* none

## Notes
* `Halt()` if the specified pointer is incorrect (i.e. was not returned by [Heap_Alloc](BVM-Heap_Alloc.md))
* `null` is NOT a valid parameter


## Example 