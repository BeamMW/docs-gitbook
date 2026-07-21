# Env::Secp_Point_mul_J

```C++
void Secp_Point_mul_J(Secp_point& dst , const Secp_scalar& s);
```
Multiplies point **J** by scalar `s` and stores result to `dst`, whereas **J** is the standard J-generator (used for the shielded output serial number)

## Parameters
* `dst ` : destination point object handle (opaque pointer)
* `s` : scalar operand

## Return value
* none

## Notes
* none

## Example 