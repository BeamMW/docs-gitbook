# Env::Secp_Scalar_neg

```C++
void Secp_Scalar_neg(Secp_scalar& dst , const Secp_scalar& src);
```
Negation: sets `dst = -src`

## Parameters
* `dst ` : destination scalar object handle (opaque pointer)
* `src` : source scalar object handle (opaque pointer)

## Return value
* none

## Notes
* `dst`, `src` don't have to be distinct

## Example 