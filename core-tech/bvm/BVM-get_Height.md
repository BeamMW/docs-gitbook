# Env::get_Height

```C++
Height get_Height();
```
Returns the current blockchain height

## Parameters
* none
## Return value
* blockchain height, **excluding** the current block being-interpreted

## Notes

Returns the number of previously mined blocks, <u>not including</u> the current block in which this function is called. The shader can read block headers from 1 up to the return value. See `get_HdrInfo()` and `get_HdrFull()`

## Example 