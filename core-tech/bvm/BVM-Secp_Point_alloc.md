# Env::Secp_Point_alloc

```C++
Secp_point* Secp_Point_alloc();
```
Allocates the point object and initializes it to 0 (a.k.a. point at infinity)

## Parameters
* none

## Return value
* point object handle (opaque pointer)
* `null` if maximum simultaneous point objects count is exceeded

## Notes
* none

## Example 