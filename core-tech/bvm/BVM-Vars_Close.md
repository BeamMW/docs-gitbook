# Env::Vars_Close

```C++
void Vars_Close(uint32_t iSlot);
```
Stops variable enumeration identified by `iSlot`

## Parameters
* `iSlot` : enumeration slot returned by [Vars_Enum](Vars_Enum)

## Return value
* none

## Notes
* the slot number should be obtained with [Vars_Enum](Vars_Enum) function

## Example 