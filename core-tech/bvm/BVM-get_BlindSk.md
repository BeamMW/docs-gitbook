# Env::get_BlindSk

```C++
void get_BlindSk(Secp_scalar& res, const void* pID, uint32_t nID, const Secp_scalar& mul, uint32_t iSlot)
```
Return blinded secret scalar gerenerated using given `ID`. Blinded result is ` sk * mul + nonce`. Nonce from `iSlot` slot is used.
This function is used in manual creation of Schnorr signature, where `mul` is Schnorr signature challange `e`, `ID` is used to generate secret key, and `nonce` is Schnorr signature nonce, which should be used only once.

## Parameters
* `res` : result scalar
* `pID` : pointer to the buffer with `id` data
* `nID` : size of the buffer
* `mul` : multiplier
* `iSlot` : iSlot


## Return value
* none

## Notes
* `iSlot` value is invalidated after this call and cannot be used afterwards

## Example

