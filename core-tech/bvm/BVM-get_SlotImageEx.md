# Env::get_SlotImageEx

```C++
void get_SlotImageEx(Secp_point& res, const Secp_point& gen, uint32_t iSlot);
```
Loads the image of the nonce from given slot. Slot should be initialized via [SlotInit](BVM-SlotInit.md)

## Parameters
* `res` : image of the nonce
* `gen` : generator point 
* `iSlot` : nonce slot number


## Return value
* none

## Notes
* none

## Example 