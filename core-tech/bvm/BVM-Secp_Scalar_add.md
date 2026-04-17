# Env::Secp_Scalar_add

```C++
void Secp_Scalar_add(Secp_scalar& dst , const Secp_scalar& a , const Secp_scalar& b);
```
Adds two scalars `a` and `b` and stores result to `dst`. Addition: sets `dst = a + b`

## Parameters
* `dst ` : destination scalar object handle (opaque pointer)
* `a` : left hand scalar operand
* `b` : right hand scalar operand

## Return value
* none

## Notes
* `dst`, `a` and `b` don't have to be distinct

## Example 