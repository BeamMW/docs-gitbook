# Env::CallFar

```C++
void CallFar(const ContractID& cid , uint32_t iMethod , void* pArgs , uint32_t nArgs , uint8_t bInheritContext);
```
Calls the contract specified by `cid`. Invokes method `iMethod` with arguments specified by `pArgs`

## Parameters
* `cid`  : id of the contact to be called
* `iMethod` : method number
* `pArgs` : the pointer to the arguments buffer
* `nArgs` : the size of the arguments buffer
* `bInheritContext` : 1 - call in the same context, 0 - create new context

## Return value
* none

## Notes
* The `pArgs` must be a valid pointer to stack memory (heap or code segments are not allowed). Will Halt() otherwise.
* The execution of the callee has full access to the stack and heap memory. Call only trusted contracts

## Example 