# Env::Secp_Scalar_alloc

```C++
Secp_scalar* Secp_Scalar_alloc();
```
Allocates the scalar object and initializes it to 0

## Parameters
* none

## Return value
* scalar object handle (opaque pointer)
* `null` if maximum simultaneous scalar objects count is exceeded

## Notes
* none

## Example 