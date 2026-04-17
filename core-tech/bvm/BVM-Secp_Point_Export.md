# Env::Secp_Point_Export

```C++
void Secp_Point_Export(const Secp_point& p , PubKey& pk);
```
Exports the point value to the **compact** form (X-coordinate and a flag for Y)
    // If the point value `p` is 0 - the pk is zeroed.

## Parameters
* `p` : point object handle (opaque pointer) to export
* `pk`: result point in a **compact** form (X-coordinate and a flag for Y)

## Return value
* none

## Notes
* If the point value `p` is 0 - the `pk` is zeroed.

## Example 