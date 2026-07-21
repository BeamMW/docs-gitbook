# Env::Secp_Point_mul

```C++
void Secp_Point_mul(Secp_point& dst , const Secp_point& p , const Secp_scalar& s);
```
Multiplies point `p` by scalar `s` and stores result to `dst`. Multiplication by a scalar: sets `dst = b * s`

## Parameters
* `dst ` : destination point object handle (opaque pointer)
* `p` : point operand
* `s` : scalar operand

## Return value
* none

## Notes
* `dst` and `p` don't have to be distinct

## Example 