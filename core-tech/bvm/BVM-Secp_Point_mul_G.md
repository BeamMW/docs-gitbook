# Env::Secp_Point_mul_G

```C++
void Secp_Point_mul_G(Secp_point& dst , const Secp_scalar& s);
```
Multiplies point **G** by scalar `s` and stores result to `dst`, whereas **G** is the standard G-generator (used for blinding factor)

## Parameters
* `dst ` : destination point object handle (opaque pointer)
* `s` : scalar operand

## Return value
* none

## Notes
* none

## Example 