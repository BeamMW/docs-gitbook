# Env::Secp_Point_Import

```C++
uint8_t Secp_Point_Import(Secp_point& p , const PubKey& pk);
```
Sets the point object `p` to the specified value `pk`, given in a **compact** form (X-coordinate and a flag for Y).

## Parameters
* `p` : point object handle (opaque pointer) to write result
* `pk`: point in a **compact** form (X-coordinate and a flag for Y)

## Return value
* 1 if the import was successful
* 0 otherwise

## Notes
* If the import is unsuccessful the point is set to 0
* if `pk` is zeroed - it's considered a valid 0 point, and the return value is 1 (i.e. successfully imported zero point)

## Example 