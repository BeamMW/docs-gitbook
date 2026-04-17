# Env::get_RulesCfg

```C++
Height get_RulesCfg(Height h , HashValue& res);
```
Sets `res` to the `Rules` hash used for the specified blockchain height, according to the consensus parameters

## Parameters
* `h` : the height at which we want to get Rules. `h` may be arbitrary (i.e. can indicate future height)
* `res` : hash of the Rules

## Return value
* the minimum height to which those Rules apply, i.e. height of the beginning of the Fork to which the specified `h` belongs. (`retval <= h`)

## Notes
* none

## Example 