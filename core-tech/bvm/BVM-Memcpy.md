# Env::Memcpy

```C++
void* Memcpy(void* pDst, const void* pSrc, uint32_t size);
```
Equivalent of standard `memmove`. Copies `size` bytes from the object pointed to by `pSrc` to the object pointed to by `pDst`.

## Parameters
* `pDst ` : pointer to the memory location to copy to
* `pSrc`  : pointer to the memory location to copy from
* `size`  : number of bytes to copy

## Return value
* `pDst`

## Notes
* none

## Example
```C++
char src[] = "This is Env::Memcmp example";
Env::Memcpy(src, src + 8, 20);
Env::DocAddText("example", src);
```
Output:
```
"example": "Env::Memcmp example"
```
