# Env::Secp_Point_neg

```C++
void Secp_Point_neg(Secp_point& dst , const Secp_point& src);
```
Negation: sets `dst = -src`

## Parameters
* `dst ` : destination point object handle (opaque pointer)
* `src` : source point object handle (opaque pointer)

## Return value
* none

## Notes
* `dst`, `src` don't have to be distinct

## Example 