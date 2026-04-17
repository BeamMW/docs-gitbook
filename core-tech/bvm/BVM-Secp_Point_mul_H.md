# Env::Secp_Point_mul_H

```C++
void Secp_Point_mul_H(Secp_point& dst , const Secp_scalar& s , AssetID aid);
```
Multiplies point **H**[aid] by scalar `s` and stores result to `dst`, whereas **H**[aid] is the generator used for asset type denoted by `aid`

## Parameters
* `dst ` : destination point object handle (opaque pointer)
* `s` : scalar operand
* `aid`: asset id 

## Return value
* none

## Notes
* none

## Example 