# Env::Secp_Scalar_mul

```C++
void Secp_Scalar_mul(Secp_scalar& dst , const Secp_scalar& a , const Secp_scalar& b);
```
Multiplies two scalars `a` and `b` and stores result to `dst`. Multiplication: sets `dst = a * b`

## Parameters
* `dst ` : destination scalar object handle (opaque pointer)
* `a` : left hand scalar operand
* `b` : right hand scalar operand

## Return value
* none

## Notes
* `dst`, `a` and `b` don't have to be distinct

## Example 