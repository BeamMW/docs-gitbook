# Env::Secp_Scalar_inv

```C++
void Secp_Scalar_inv(Secp_scalar& dst , const Secp_scalar& src);
```
Inverts scalar `src` and stores result to `dst`. Invertion: sets `dst = src^-1;`

## Parameters
* `dst ` : destination scalar object handle (opaque pointer)
* `src` : destination scalar object handle (opaque pointer)


## Return value
* none

## Notes
* `dst`, `src`  don't have to be distinct
* if `src == 0`, then `dst 👍 = 0`
* Otherwise `src * dst == 1`

## Example 