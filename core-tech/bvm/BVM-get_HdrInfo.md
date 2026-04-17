# Env::get_HdrInfo

```C++
void get_HdrInfo(BlockHeader::Info& hdr);
```
Fills the structure `hdr` with the appropriate header info

## Parameters
* none

## Return value
* none

## Notes
* Must set `hdr.m_Height` before invocation
* `Halt()` if the specified height is invalid

## Example 