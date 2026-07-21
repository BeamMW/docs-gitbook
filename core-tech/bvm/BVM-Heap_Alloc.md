# Env::Heap_Alloc

```C++
void* Heap_Alloc(uint32_t size);
```
Allocates the specified `size` on heap, and returns its pointer if allocation is successful.
    
## Parameters
* `size` : number of bytes to allocate

## Return value
* pointer to allocated region
* `null` otherwise (if no more heap available)

## Notes
* `size == 0` is a valid allocation size as well


## Example 