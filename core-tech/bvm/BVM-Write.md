# Env::Write

```C++
void Write(const void* pData, uint32_t nData, uint32_t iStream);
```
Writes data to output stream. 

## Parameters
* `pData` : pointer to data buffer
* `nData` : size of data buffer in bytes
* `iStream` : stream number to write. Possible values: 
    * `Shaders::Stream::Out`(0) : standard output stream
    * `Shaders::Stream::Error`(1) : standard error stream


## Return value
* none

## Notes
* none

## Example 
