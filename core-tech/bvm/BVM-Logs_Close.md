# Env::Logs_Close

```C++
void Logs_Close(uint32_t iSlot);
```
Stops log enumeration identified by `iSlot`

## Parameters
* `iSlot` : enumeration slot returned by [Logs_Enum](Logs_Enum)

## Return value
* none

## Notes
* the slot number should be obtained with [Logs_Enum](Logs_Enum) function

## Example 