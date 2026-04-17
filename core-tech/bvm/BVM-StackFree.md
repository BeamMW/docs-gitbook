# Env::StackFree

```C++
void StackFree(uint32_t size);
```
Frees the stack memory allocated previously by [StackAlloc](StackAlloc).
    
## Parameters
* `size` : number of bytes to free

## Return value
* none

## Notes
* Not necessarily to call, the extra stack is freed automatically on function return
* If invalid `size` is specified - stack corruption or `Halt()` may occur


## Example 