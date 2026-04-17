# Env::Secp_Scalar_import

```C++
uint8_t Secp_Scalar_import(Secp_scalar& s , const Secp_scalar_data& data);
```
Sets the scalar object to the specified value. If the specified value is invalid (not less than the group order) - its valid is normalized (group order is subtracted).

## Parameters
* `s` : scalar object handle (opaque pointer)
* `data` : value

## Return value
* 1 if the normalization was not necessary
* 0 otherwise.

## Notes
* none

## Example 